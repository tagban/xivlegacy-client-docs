# XIVLegacy client documentation

Notes on how the FINAL FANTASY XIV 1.23b client behaves, gathered while building a native
macOS client for it and running it against a local [Garlemald](https://github.com/swstegall/Garlemald-Server)
server: the network protocol and opcodes, the login and zone-in flows, the tutorials, graphics and
input findings, and a dated log of everything we ran into.

This repository is documentation only. It contains no game files, no game text and no code from
the client, and it doesn't describe how to rebuild the native client.

> Not affiliated with Square Enix. FINAL FANTASY XIV is a registered trademark of Square Enix Holdings Co., Ltd.
> Also not to be confused with the **XIVLegacy** GitHub organization (the
> [xivl-captures](https://github.com/XIVLegacy/xivl-captures) research project), a separate team
> whose CC BY 4.0 data some of these pages credit.

## Contents

| Page | What it covers |
| --- | --- |
| [Overview](docs/overview.md) | What the native client is, status, platforms |
| [Architecture](docs/architecture.md) | Recompiled client, portable Windows runtime, threads and the guest lock |
| [Protocol](docs/protocol.md) | Servers and ports, packet framing, compression, the login, zone-in and event flows |
| [Opcode reference](docs/opcodes.md) | Every known game-message opcode: name, handler, evidence, how often we saw it |
| [Graphics](docs/graphics.md) | Direct3D 9 on Metal, shader translation, the cutscene clear bug, stutter |
| [Input, audio and files](docs/input-audio-files.md) | How input reaches the game, XAudio2, the script file name cipher, install layout |
| [Tutorials and gameplay](docs/gameplay.md) | The Gridania opening step by step, combat, targeting, actions, what each class needs |
| [Testing](docs/testing.md) | Driving the real client unattended, and the class test matrix |
| [Findings log](docs/findings.md) | Every bug and lesson, dated |

Server-side notes for our Garlemald instance live in
[xivlegacy-server-docs](https://github.com/tagban/xivlegacy-server-docs).

## Regenerating the opcode table

```sh
python3 tools/gen_opcodes.py --garlemald <Garlemald checkout>/map-server/src/packets/opcodes.rs
```
