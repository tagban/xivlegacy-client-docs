# Tutorials and gameplay

What the client expects at each step, observed by driving the real client against Garlemald.
Server-side script details are in the
[server docs](https://github.com/tagban/xivlegacy-server-docs).

## The Gridania opening (Sundered Skies), step by step

A brand-new character (play time 0) logs in at the opening spot in the Central Shroud.

1. **Controls question:** keyboard and mouse, or gamepad. It appears 10–60 s after zone-in, over
   a black screen.
2. **Intro cutscene:** several minutes long, with a long static shot near the end. It finishes
   with Yda's "Ugh…" dialog. Press Enter to advance.
3. **Camera tutorial:** "Moving the Camera". The tutorial waits for the camera keys (I J K L) or
   a right-drag. Movement is locked during this step, so W does nothing.
4. **Main menu and journal tutorials:** "The Main Menu" (Home), then "Journal". The client stays
   in a restricted tutorial mode, with no movement and no talking, until the journal is opened
   and a quest is picked: Home → Journal → click the quest.
5. **Getting Around:** "Move to Yda". Walking into Yda's trigger circle starts her push event.
   If you're already inside the circle, step out and back in.
6. **Speaking to friendly NPCs:** with Yda targeted, Enter opens the talk.
7. **Papalymo:** talk to him. He's to the side, and Tab only cycles targets in front of you, so
   turn first.
8. **Yda again:** the warp into the combat area.

### Combat tutorial

- **Intro:** Yda and Papalymo talk ("I hope you came ready for a fight!").
  - **Disciples of War:** the intro ends with the "Active & Passive Modes" tip. Press **F**.
  - **Disciples of Magic:** the intro is longer (about three minutes) and only returns once the
    player presses **F**.
- **Targeting lesson:** "Place your targeting cursor on one of the bloodthirsty wolves". Confirm
  the tip, then target a wolf (click or Tab).
- **The fight:** Enter on the wolf locks on and starts auto-attack. Actions are a hotbar key plus
  Enter to confirm.
  - War classes get tips in order: a successful attack, TP, then a weaponskill once TP reaches
    1,000.
  - Yda and Papalymo fight alongside you and can kill the wolves before you've used a weaponskill.
- **After the battle:** a closing cutscene (several minutes) and an item dialog, then the
  hand-off to Gridania.

### Class differences

| Classes | Tutorial path |
| --- | --- |
| GLA, PUG, MRD, ARC, LNC | War branch: F → targeting lesson → attack, TP and weaponskill tips → battle complete |
| THM, CNJ | Magic branch: longer intro → targeting lesson → defeat the enemies |
| Crafters and gatherers | No fight at all: the combat director has no branch for them, and they start with no gear |

## Combat facts

- **Active mode first:** auto-attack only runs in active mode, so draw the weapon with F.
- **Range:** melee is about 3 yalms. Out of range, the server keeps the lock but skips swings.
- **Tutorial wolves:** 250 HP each.
- **Open-world HP (retail captures):** level 1 = 80, level 2 = 89, level 3 = 99, level 10 ≈ 175,
  level 20 ≈ 410, level 30 ≈ 750.
- **Starting HP:** a level-1 Gladiator has 280 HP and 25 MP.
- **Visibility:** the server sends actors within 50 yalms. Anything further away doesn't exist for
  the client, so NPCs and monsters can pop in late.

## Chocobo rental

Talking to a chocobo lender opens a welcome line, then the main menu, and renting costs 800 gil.
The menu's "has funds" flag comes from the server. Choosing to rent mounts the player (music 64)
and warps them just outside the city.
