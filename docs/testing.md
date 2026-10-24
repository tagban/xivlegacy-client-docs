# Testing

## Driving the real client unattended

The native client accepts commands on a named pipe. Each line is one command:

| Command | Effect |
| --- | --- |
| `key <name>` | Press and release a key (SDL key names: `Return`, `Tab`, `f`, `Home`, `Escape`, `1`…) |
| `hold <key> <ms>` | Hold a key (walking, camera) |
| `click <x> <y>` / `move <x> <y>` | Mouse click or move, in window coordinates (1280×720) |
| `type <text>` | Type text |
| `snap <png>` | Save the current frame |

A test run combines three things:

- **Input** through the pipe.
- **The server log as the referee:**
  - every event start and end, Run Event Function and End Event;
  - target change, auto-attack and kill;
  - zone change and signal.
- **Screenshots** for what only the screen shows. The yellow class icon at the bottom left means
  the HUD is up, i.e. a cutscene is over; the controls question has a recognizable keyboard
  picture.

Test characters are reset in the server database between runs, as brand-new characters (play
time 0, no items, quests or hotbar) of the class under test. That makes each run a real first
login: starter kit, opening quest and intro cutscene.

## What a run checks

**Full run (Gridania opening):**
1. Login and zone-in; new-player setup.
2. The intro cutscene finishes.
3. The camera and journal tutorials.
4. Yda's walk-up trigger; talks with Yda and Papalymo.
5. The warp into the combat area.
6. The combat intro, then F draws the weapon.
7. The targeting lesson.
8. The player deals damage; the battle completes.
9. The tutorial ends and hands off to Gridania.

**Field run (Central Shroud):** a new character of the class starts at a Star Marmot camp. It
must have its class weapon equipped, draw with F, engage a monster and kill it.

## Results (October 24, 2026)

| Class | Opening + combat tutorial | Open-world fight |
| --- | --- | --- |
| Gladiator | Pass | Pass (1 kill) |
| Pugilist | Pass | Pass (1 kill) |
| Marauder | Pass | Pass (1 kill) |
| Archer | Pass | Pass (2 kills) |
| Lancer | Pass | Pass (2 kills) |
| Thaumaturge | Server flow correct; automated run pending | Pass (2 kills) |
| Conjurer | Server flow correct; automated run pending | Pass (2 kills) |
| Crafters and gatherers | No combat tutorial exists for them | Not applicable (no starter gear) |

Problems these runs found are in the [findings log](findings.md) for October 24.

## Pitfalls for test scripts

- **Wait on screen or server state, not fixed times:** the intro and closing cutscenes vary
  between three and nine minutes.
- **Tab isn't nearest-first:** turn toward the target, or keep cycling until a lock actually lands
  swings.
- **Actions need a target confirm:** press the hotbar key, then Enter.
- **The magic combat intro waits for F.**
- **Don't warp a character mid-cutscene:** the tutorial's own closing warp lands afterwards and
  undoes yours.
