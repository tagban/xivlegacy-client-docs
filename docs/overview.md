# Overview

## What it is

FINAL FANTASY XIV 1.23b (game version `2012.09.19.0001`, the last 1.x build) running as a native
program, without Wine. The game executable from the player's own install is translated to native
code on their machine, and runs against a small portable implementation of the Windows APIs it
uses. Direct3D 9 is drawn through Metal. Nothing from Square Enix is shipped: the player supplies
their own disc image and patches.

The server side is [Garlemald](https://github.com/swstegall/Garlemald-Server), a Rust port of
Project Meteor, run locally. See [xivlegacy-server-docs](https://github.com/tagban/xivlegacy-server-docs).

## Platforms

macOS (Apple silicon) comes first. Linux is the likely second, with a Vulkan back end. Windows can
run the original client directly, so it comes last.

## Status (October 2026)

- **Working:**
  - logging in through a web sign-in, the lobby, character creation and character select;
  - zoning into all three opening cities, with cutscenes, sound, walking and NPC conversations;
  - the full Gridania opening and combat tutorial for all five Disciple of War classes;
  - open-world fights for all seven combat classes;
  - quests, the journal, chocobo rental and the menus tested so far.
- **Performance:** about 50-55 fps walking in the world on an Apple M4, after the stutter work
  in [Graphics](graphics.md).
- **Launcher:** a small macOS app (sign-in, server choice, remembers the account in the Keychain)
  that starts the local server if needed and runs the game.

## Pages

- [Architecture](architecture.md)
- [Protocol](protocol.md) and the [opcode reference](opcodes.md)
- [Graphics](graphics.md)
- [Input, audio and files](input-audio-files.md)
- [Tutorials and gameplay](gameplay.md)
- [Testing](testing.md)
- [Findings log](findings.md)
