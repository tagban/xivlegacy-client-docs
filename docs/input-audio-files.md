# Input, audio and files

## Input

- **Subclassing:** keys and the mouse reach the game through `SetWindowSubclass` (COMCTL32
  ordinals 410–413) on its window. The window procedure itself ignores input.
- **The pointer:** the game places it with `GetCursorPos`.
- **Text fields** need `WM_CHAR` for Backspace, Tab, Enter and Escape. `TranslateMessage` normally
  produces those, and SDL's text input leaves them out.
- **Controls question:** the client asks keyboard-and-mouse or gamepad on first play. The gamepad
  option is greyed out until DirectInput/XInput report a pad.

### Default keyboard behavior (1.23b)

- **Movement:** W A S D. **Camera:** I J K L, or the right mouse button.
- **Targeting:** Tab cycles targets in front of the player, not nearest-first. A click targets
  directly.
- **Enter:**
  - on a friendly target: talk;
  - on an enemy target: lock on, which starts auto-attack in active mode;
  - in dialog: advance.
- **F** toggles active mode (draws the weapon).
- **Actions take two keys:** pressing a hotbar key (1–0) puts the action into target selection,
  and Enter confirms it.
- **Home** opens the main menu. **Escape** backs out of menus. Extra Escapes can leave keyboard
  focus on the action bar, which then swallows F.

## Audio

XAudio2 2.4 on top of SDL3 audio. The engine fills the device queue in 50–80 ms batches whenever
the guest lock is free, and asks for the lock urgently only when the queue runs low.

## Files

### Install layout

- **The disc** (July 2010, InstallShield cabinets) holds about 137,800 files, 9.7 GB:
  - the `NewComponent1` group → `client/`;
  - `NewComponent2` → `data/`;
  - five launcher programs → the install root.
- **Patches:** 52 ZIPATCH files take it from `2010.09.18.0000` (boot) to `2012.09.19.0001` (game).
  The result is 188,697 files, 12 GB.
- **Version files:** patches don't update `boot.ver`/`game.ver`, but the retail updater did, and
  servers expect `2012.09.19.0001`.

### ZIPATCH format

PNG-style chunks after a 12-byte `\x91ZIPATCH\r\n\x1a\n` signature: `size u32 BE, tag[4], body,
crc32 BE`.
- **`FHDR`, `APLY`, `APFS`:** metadata.
- **`ADIR` / `DELD`:** create / delete a directory.
- **`ETRY`:** entries with mode (`0x41` add, `0x4D` modify, `0x44` delete), source and destination
  SHA-1, and compression (`0x5A` zlib, `0x4E` stored).
  - The data is always the **whole new file**, not a diff.
  - `src_sha1` isn't a reliable link to the previous version; `dst_sha1` is.

### Client script file names

Client scripts live under `client/script/` as `.le.lpb` files. Their paths are obscured with a
simple letter substitution:

- `a`–`j` ↔ `9`–`0`
- `k`–`z` ↔ `z`–`k` (reversed)

For example, `7vxx9w6` = `command`, `658p3` = `debug`, and
`729s9/wu7/xvwrq5s/...` = `chara/npc/monster/...`.

Decoding the folder shows which class scripts exist. There are 804 monster scripts, which servers
need: a monster actor class has to name a script the client has, or the client fails to spawn it.

### Folders the game needs

`Documents\My Games\FINAL FANTASY XIV` has to be creatable: the folders above a mount point must
exist.
