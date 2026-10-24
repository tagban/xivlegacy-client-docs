# Findings log

Newest first. "Client" means the native client or how the 1.23b client behaves; "server" means
Garlemald (details in the [server docs](https://github.com/tagban/xivlegacy-server-docs)).

## 2026-10-24

- **Server: the war-class combat tutorial could softlock.**
  - The tutorial waits for an attack, then TP 1,000, then a weaponskill, then battle complete.
  - Yda and Papalymo can kill the wolves first, and the tutorial then waited for a weaponskill
    forever.
  - Fixed: pending milestones are released when the battle completes.
- **Server: crafters and gatherers get nothing at first login.**
  - They get no starter items, and the combat tutorial has no branch for them.
  - Not fixed.
- **Client: the magic combat intro only returns after F.** Test scripts have to press F during it.
- **Client: tutorial input locks.** The opening locks movement and talking until each tutorial
  step is done: the camera, then the journal (open it and pick the quest).
- **Client: hotbar actions take two keys.** Press the action, then Enter to confirm the target.
- **Client: Tab cycles targets in front of the player**, not nearest-first.
- **Server: the chocobo rental never worked.** Two causes:
  - Every NPC script read the player's gil as 0, so the "has funds" flag was always false.
  - The active quest's talk handler sent an End Event right after the lender's welcome line,
    closing the conversation, so the menu's answer never came back.
  - Both fixed.
- **Server: pop-in.** Actors are only sent within 50 yalms and never removed when out of range,
  so NPCs (the chocobo lender) and monsters appear late.
- **Server: the open world had no monsters.** We added about 3,200 from retail captures and camp
  maps, approximate where no exact position exists.
- **Client: monsters crashed the client.** A monster class with no server-side init script was
  sent the townsperson init values, and the client raised an exception in DepictionJudge. Fixed
  with the standard monster values.
- **Server: monster HP.** Gamer Escape's HP table mixes level brackets (a level-1 marmot listed at
  720). Retail captures give 80 at level 1.

## 2026-10-23

- **Server: Thaumaturge/Conjurer combat tutorial softlock.** The magic branch reopened the
  tutorial event and never closed it. The client stayed in event mode: no F, no attacks. Fixed;
  submitted upstream as Garlemald PR #212.
- **Client: walking and loading stutter.**
  - Synchronous shader and pipeline compiles caused hitches, and so did other threads holding
    long lock turns.
  - Fixed with background compiles and drawer-only long turns: hitches went from 15–19 to 1–2 per
    walk.
- **Client: combat felt off.** A click selects; auto-attack needs lock-on (Enter or a second
  click) in active mode.
- **Client: cutscenes smeared** into radial streaks and doubled characters. Deferred clears were
  dropped when the game switched render targets before drawing. Fixed.

## 2026-10-22

- **Client: first full login.** Character creation through the real UI, zoning onto the Limsa
  Lominsa boat, walking around, with sound.
- **Client: input.** Input arrives through window subclassing, not the window procedure. Text
  fields need `WM_CHAR` for control keys.
- **Client: the lobby needs a 2012 clock.**
- **Client: Metal details.** No negative base vertex; a half-pixel offset relative to D3D9.

## Open questions

- A periodic ~150 ms hitch about 100 s into play, while the drawing thread is busy with its own
  work.
- Clicking an NPC twice while a tutorial box is up (a type-51 talk) can leave the client waiting;
  only a relog fixes it.
- The gamepad is greyed out on the controls screen.
- Ground heights for monsters away from captured positions: the server has no terrain data.
