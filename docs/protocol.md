# Protocol

The 1.23b client talks to four servers. With Garlemald all four run locally:

| Server | Port (Garlemald default) | Role |
| --- | --- | --- |
| Web | 54993 | Sign-in and sign-up pages; returns the session id |
| Lobby | 54994 | World list, character list, character creation, picking a character |
| World | 54992 | Session routing, groups, chat relay between map servers |
| Map | 1989 | The game itself: actors, movement, events, combat, inventory |

The client asks for `lobby01.ffxiv.com`, which a client or hosts file has to point at the lobby
server.

## Sign-in

1. `POST /login` on the web server with `username` and `password`.
2. It answers with a redirect to `ffxiv://login_success?sessionId=<56 hex characters>`, or back to
   the form with `err=...`.
3. The session id is the game's launch argument. The title screen then logs in to the lobby on its
   own.

## Framing

Every TCP message is a **base packet** holding one or more **subpackets**. All integers are
little-endian.

**Base packet header** (16 bytes):

| Offset | Size | Field |
| --- | --- | --- |
| 0 | u8 | is_authenticated |
| 1 | u8 | is_compressed (the subpacket area is zlib-compressed when set) |
| 2 | u16 | connection_type |
| 4 | u16 | packet size, header included |
| 6 | u16 | subpacket count |
| 8 | u64 | timestamp (ms) |

**Subpacket header** (16 bytes):

| Offset | Size | Field |
| --- | --- | --- |
| 0 | u16 | subpacket size, header included |
| 2 | u16 | type: 3 = game message; 7/8 = ping/pong; others are session control |
| 4 | u32 | source actor id |
| 8 | u32 | target actor id (the world server drops frames with target 0) |
| 12 | u32 | unknown |

**Game-message header** (16 bytes, type 3 only), followed by the body:

| Offset | Size | Field |
| --- | --- | --- |
| 0 | u16 | 0x14 |
| 2 | u16 | **opcode** (see the [opcode reference](opcodes.md)) |
| 4 | u32 | 0 |
| 8 | u32 | timestamp (s) |
| 12 | u32 | 0 |

Lobby traffic is additionally Blowfish-encrypted with a key set up during its handshake.

The same opcode number can mean different things by direction. For example, `0x00CA` is the
player's position update going up and Add Actor coming down, and `0x00CC` is Target Locked going up
and Actor Instantiate coming down.

## Actor ids

- **Players:** their character id.
- **Zone actors:** `(4 << 28) | (zone << 19) | number`.
  - Populace NPCs use small numbers (1, 2, 3…).
  - Garlemald marks content-instance actors with bit `0x40000`, and our open-world monsters start
    at `0x20000`.
- **Static command actors:** `0xA0F00000 | command id`. For example, `0xA0F05209` is the
  active-mode toggle and `0xA0F05E93` is the quest journal.
- **Directors:** `0x6...`-prefixed actors that run quests and content scripts.

## Zone-in

1. The server sends Set Map (`0x0005`), music and weather.
2. Then a mass delete of old actors (`0x0006`/`0x0007` start/end with `0x0008`–`0x000B` bodies
   between them).
3. Then the zone-in bundle: the player, nearby actors (Add Actor `0x00CA`, Actor Instantiate
   `0x00CC`, Set Actor Position `0x00CE`, state, appearance and properties) and any director with
   its event kick.
4. The client answers Language Code (`0x0006`) when it's ready for world spawns, and Zone-In
   Complete (`0x0007`) once loaded.
5. After that the server streams actors as the player moves: the player's position (`0x00CA`) up,
   newly in-range actors down.

The client's own loading takes a while: about 40 seconds from zone-in to the first movement update
in Gridania on our test machine.

## Events (talking, cutscenes, menus)

| Opcode | Direction | Meaning |
| --- | --- | --- |
| `0x012D` Event Start | up | The client starts an event on an actor: `talkDefault` (click/Enter on an NPC), `pushDefault` (walked into a trigger), `noticeEvent` (answer to a server kick), `commandDefault` (hotbar action), `commandForced` (the F active-mode toggle), `commandRequest` (journal and similar) |
| `0x012F` Kick Event | down | The server asks the client to start an event (cutscenes, director steps) |
| `0x0130` Run Event Function | down | Runs a client-side script function inside the open event (dialog, cutscene, menu) |
| `0x012E` Event Update | up | The function's result: menu choice, dialog finished, typed name |
| `0x0131` End Event | down | Closes the event; the client is free again |

Rules we learned the hard way:

- **One event at a time.** While an event is open, the client blocks movement, talking, F (active
  mode) and attacks.
  - An event that's never ended leaves the player stuck. This was the cause of the
    Thaumaturge/Conjurer combat tutorial softlock.
  - An End Event sent while the NPC's script is still mid-conversation closes the conversation
    early, and the menu that follows never answers. This was the cause of the chocobo rental bug.
- **A Run Event Function needs an open event.** One sent with an empty event name or owner is
  silently ignored by the client.
- **Event type 51** is a talk made while a push event from the same NPC is still open.

## Combat

| Opcode | Direction | Meaning |
| --- | --- | --- |
| `0x00CD` Target Selected | up | The current target and, if auto-attack is on, the attack target (`0xE0000000` means none) |
| `0x00CC` Target Locked | up | Lock-on (a second Enter on the target); turns auto-attack on |
| `0x0134` Set Actor State | down | Passive/active/dead etc. Active mode is state 2 |
| `0x012D` + `commandForced` on `0xA0F05209` | up | F: toggle active mode (draw or sheathe the weapon) |
| `0x012D` + `commandDefault` on `0xA0F0xxxx` | up | A hotbar action (`xxxx` = command id) |

The server only accepts auto-attack in active mode, and only within range (melee about 3 yalms).
