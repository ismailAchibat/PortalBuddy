# PortalBuddy - project handoff (state at end of session 2, 2026-11-06)

## 1. Context

- **Course**: *Realite etendue et interactions multimodales - Unreal Engine* (ENSIM, Corentin Coupry, 2026 edition).
  Solo project, about 6 weeks. The single hard requirement: **one innovative mechanic**.
- **Grading criteria of the first session** (all targeted): innovative mechanic, playable proof of concept,
  versioning (repo, clear commits, identifiable first version).
- **Machine**: university PC (network drive `H:`, no admin rights assumed). UE 5.6.1, **Blueprint only**
  (no C++), editor UI in **French**. Engine path: `D:\PROGRAMMES\Epic Games\UE_5.6`.
- **University proxy**: `http://proxy.univ-lemans.fr:3128` (Squid). It returns 403 for `127.0.0.1` unless
  `NO_PROXY=127.0.0.1,localhost,::1` is set (it was set as an environment variable in session 2).
- **Project folder**: `H:\Home\Bureau\UnrealProjects\Projet-unreal-engine\PortalBuddy`
- **Repo**: https://github.com/ismailAchibat/PortalBuddy (user: ismailAchibat).

## 2. The game

FPS puzzle. The player can switch the view to a small robot, **BP_Buddy**, which is more agile.
The robot shoots a marble; when it hits the **floor**, a portal opens and the **player body teleports**
there (animated: a portal forms under the player, he sinks in, comes out of the exit portal, then control
returns to him). While the robot is controlled, the player body stays where it is (and is meant to be
vulnerable to future enemy bots). Goal: cross every room and reach the arrival pad.

Course UI term mapping: Content Drawer = Tiroir a contenu, Details = Details, Outliner = Organiseur,
Place Actors = Placer des acteurs, Event Graph = Graphe d'evenements, Compile = Compiler, Play = Jouer.

## 3. Git state

| Ref | Meaning |
|---|---|
| `main` | tag `v0.3` = commit `e6b87d7` (robot, switch, portal with entry+exit animation, crosshair, physics fix) |
| tags | `v0.1-poc` (switch only), `v0.2-portal`, `v0.3` |
| `feature/puzzle-rooms` | commit `b660016`, pushed, **not merged** into `main`: 4 puzzle rooms + checkpoints + pause menu |

Use `git -c safe.directory=* ...` on the network drive. Commit message style used so far: short imperative
title, bullet list, then the `Co-Authored-By` line from the session's system reminder.
The working tree was clean after the last push (only this handoff and a `CLAUDE.md` update may be new).

## 4. Assets and how they work (all under `Content/`)

### Input (`Content/Input`)
`IMC_Default` maps: WASD/arrows/gamepad -> `IA_Move`; Space -> `IA_Jump`; gamepad -> `IA_Look`;
**E -> `IA_SwitchView`**; **Left mouse -> `IA_Fire`**; **Escape and Tab -> `IA_Pause`**
(`IA_Pause` has `bTriggerWhenPaused = true` so it can also close the menu while paused).
`IMC_MouseLook` maps Mouse2D -> `IA_MouseLook`. The player controller adds both contexts.

### Player: `FirstPerson/Blueprints/BP_FirstPersonCharacter` (template, modified)
- `IA_SwitchView` Started -> `GetActorOfClass(BP_Buddy)` -> `Possess` via `GetPlayerController(0)`.
- Variable `RespawnLocation` (Vector, set on BeginPlay, updated by checkpoints).
- `bRunPhysicsWithNoController = true` (otherwise an unpossessed Character freezes in mid-air and keeps
  its run animation; this was a real bug the user reported).
- Controller `BP_FirstPersonPlayerController`: `IA_Pause` toggles `WBP_PauseMenu` (variable `PauseRef`,
  `SetGamePaused`, input mode UI/Game, mouse cursor).

### Robot: `Buddy/BP_Buddy` (Character)
- Capsule HalfHeight 40, Radius 25. Components: `BuddyBody` (sphere, scale 0.4, no collision,
  Owner No See), `BuddyCamera` (Use Pawn Control Rotation, at x=25, z=10).
- Movement: walk 900, jump Z 800, air control 0.6, max accel 3000, **BrakingDecelerationFalling 1800**
  (so releasing keys stops it in the air and landings on ledges are precise),
  `bRunPhysicsWithNoController = true`, `AutoPossessAI = Disabled`.
- Event Graph: move/look/mouse-look/jump (Vector2D `ActionValue` needs a Break Vector2D node);
  `IA_SwitchView` -> blocked if a `BP_Portal` exists, else Possess the player;
  `IA_Fire` -> blocked while `bCooldown` or while a portal exists, else spawns `BP_BuddyShot` at
  camera + 60 cm forward with the control rotation, then 1.5 s cooldown;
  `Possessed` creates and shows `WBP_Crosshair`, `Unpossessed` removes it (variable `CrosshairRef`).

### Marble: `Buddy/BP_BuddyShot`
Sphere root `CollisionComponent` (radius 8, profile Projectile), `ProjectileMovement` speed 4000, no
gravity, lifespan 4 s. `OnComponentHit`: if `ImpactNormal.Z > 0.7` (floor) spawn `BP_Portal` at
`ImpactPoint` and destroy; otherwise just destroy (walls and ceilings never get a portal).

### Portal: `Buddy/BP_Portal` (exit and entry are the same Blueprint)
- Components: `PortalDisc` (flat cylinder with engine material `EmissiveMeshMaterial`, tinted blue by a
  dynamic material instance, parameter `Color`), `PortalLight`.
- Variable `IsEntry` (public, **expose on spawn**). Entry portal: only plays `EntryLifeTL` (scale 0 -> 1 -> 0
  over 2.8 s) then destroys itself.
- Exit portal (IsEntry false) timeline of the whole sequence:
  t=0 find the player, `BaseLoc` = player location minus 98 cm in Z, spawn the entry portal there,
  play `LifeTL` (4 s: forms in 1 s, closes at 3.4-4 s);
  t=1 s disable player movement and collision, `bSinking = true`, `EmergeTL` **reversed** (the player
  sinks 1 -> 0: moves down 188 cm and scales down to 5 %);
  `EmergeTL` finished while sinking -> `BaseLoc` = exit portal location, play `EmergeTL` forward;
  at the end re-enable collision, `SetMovementMode(Walking)`, `Possess` the player.
- Because the player is only possessed at the end, there is never more than one portal sequence at a time.

### Puzzle rooms (branch `feature/puzzle-rooms`)
Level `Content/Levels/Lvl_PortalBuddy` (the old template level `FirstPerson/Lvl_FirstPerson` still exists).
Whitebox from `/Engine/BasicShapes/Cube` with the template grid materials; coloured point lights per room.
Coordinates in cm, floor top at z=0, X is the walking direction, rooms are 2000 long and 1200 wide.

| Room | X range | Idea | Robot-only constraint |
|---|---|---|---|
| 1 The Wall | 0-2000 | 180 cm high, 300 thick wall at x=1000-1300 | player jump apex is about 90 cm, robot about 326 cm |
| 2 The Duct | 2000-4000 | wall at x=3000 with a 110x100 cm tunnel at y=370-515 containing the pressure plate (3000,440); plate opens `Door_R2` at (4000,0,175) | tunnel is too low for the player capsule |
| 3 The Archipelago | 4000-7200 | void with start island, 2 stones, end island, gaps of 650 cm | player cannot cross a 650 cm gap |
| 4 The Tower | 7200-9200 | ledges (tops 180/360/540) then a 720 cm tower block with the `Goal` pad at (9025,0,720) | steps of 180 cm |

Gameplay actors (outliner folder `Gameplay`): `PlayerStart` (300,0,110), `BP_Buddy_Test` (380,120,48),
`CP0_Start` (silent), `CP1_AfterRoom1` (2250), `CP2_AfterRoom2` (4200), `CP3_AfterRoom3` (7400),
`KillZone_Void`, `Plate_R2`, `Door_R2`, `Goal`.

| Blueprint | Role |
|---|---|
| `BP_Checkpoint` | overlap with the player: sets his `RespawnLocation` (checkpoint + 110 in Z), shows `WBP_Toast` 2.5 s. Variable `Silent` skips the banner |
| `BP_KillZone` | player -> teleported to `RespawnLocation`; robot -> teleported next to the player |
| `BP_PressurePlate` | public `DoorRef`; the robot (cast to `BP_Buddy`) presses it once, light turns green, calls `OpenDoor` |
| `BP_Door` | `OpenDoor` custom event, `DoorTL` slides it up 400 cm, status light red -> green |
| `BP_GoalTrigger` | player overlap: shows `WBP_Victory`, pauses |
| `WBP_PauseMenu` | rules in French + Resume/Quit |
| `WBP_Victory` | "BRAVO !" + Replay (`OpenLevel Lvl_PortalBuddy`) / Quit |
| `WBP_Toast` | checkpoint banner |
| `WBP_Crosshair` | cyan `+` at screen centre, aligned with the marble's origin |

## 5. What was verified (in Play In Editor, with simulated input) and what was not

Verified: view switch both ways; robot movement and crosshair visibility; portal sequence with entry and
exit animation (player position and scale sampled over time); unpossessed physics fix; robot jumps onto the
room-1 wall and lands on it; robot crosses the room-2 tunnel and the plate opens the door; falling in the
void respawns at the checkpoint; pause menu opens and closes with Tab; goal shows the victory screen;
screenshots of rooms 1-4, the pause menu and the victory screen looked correct.

Not verified: how the rooms feel when played by a human (difficulty, jump distances in rooms 3 and 4);
the checkpoint banner on screen; tinted pad colours in game; that a full playthrough works end to end.
**The user is going to play the rooms and give feedback: that is the very next input to expect.**

## 6. Decisions and gotchas worth remembering

- The user first wanted to learn Blueprints by hand, but in session 2 explicitly asked Claude to build
  everything. Keep explaining what was built, but do the work.
- Portal only on **floors** (normal Z > 0.7), **one portal at a time**, robot can't fire while a portal exists.
- Robot is not given a separate "dead" state; falling is handled by `BP_KillZone`.
- Unreal / MCP gotchas:
  - Stop Play In Editor before compiling Blueprints or creating assets (`PIE_ACTIVE` error).
  - `create_level` refuses with `DIRTY_PACKAGES` unless `saveDirtyPackages: true`.
  - `create_material` needs consent the user refused; use `/Engine/EngineMaterials/EmissiveMeshMaterial`
    (additive, vector parameter `Color`) through a dynamic material instance for glowing parts.
  - `edit_graph` batches are all-or-nothing; connecting an exec output replaces its previous link;
    deleting nodes/links is not available in batches; existing nodes are referenced by `nodeGuid` in
    `fromNodeId`/`toNodeId`; cast output pin is named `AsBP <Name>` (e.g. `AsBP Buddy`);
    `ActionValue` Vector2D pins need `BreakStruct /Script/CoreUObject.Vector2D` (pins `InVec`, `X`, `Y`);
    use `MathExpression` with `VECT(a,b,c)` for vector math; a Timeline node has `Play`,
    `PlayFromStart`, `ReverseFromEnd`, `Update`, `Finished` and one pin per track.
  - Reading another Blueprint's variable/function: use `VariableGet`/`CallFunction` with `memberClass` set to
    the `/Game/...BP_X.BP_X_C` path.
  - Widgets: `add_panel_widget` with `widgetKind: widget_tree` builds a whole screen; `add_event` with
    `componentName` + `eventName: OnClicked` binds buttons; write accents as `\u00e9` escapes in JSON.
  - Esc stops Play In Editor; use Tab for the pause menu there.
  - `control_actor.spawn` batch replies are very long (the `report` option did not shorten them);
    the `mcp.placement.ok` tag silences intentional overlaps.
  - Testing tools that worked: `control_editor.play`, `simulate_input` (key_tap, key_down + holdSeconds,
    `widget_list`), `control_actor.get_transform` with `readMode: motion` (+ `inputs`) to sample movement,
    `control_editor.screenshot` (editor viewport with `location`/`rotation`, or `full_editor_window` to see UMG)
    then read the PNG from `Saved/Screenshots`.
- Known cosmetic point: the engine material grid texture is visible on the portal disc and pads.

## 7. Suggested next steps (ask the user which)

1. **Collect the user's feedback on rooms 1-4**, then tune (jump gaps, plate visibility, signs, difficulty).
2. Merge `feature/puzzle-rooms` into `main` when the user is happy (ask first), tag it (`v0.4`).
3. Crosshair that turns green over a valid floor and red otherwise (offered, not built).
4. Defender bots: stationary turret-type enemies that shoot the player's body (not the robot), player health,
   game over and respawn at the last checkpoint (planned for week 5).
5. Set `Lvl_PortalBuddy` as the project's default map (Project Settings > Maps & Modes), so a packaged build
   starts there.
6. Polish: sounds (portal, checkpoint, door), a custom portal material if the user allows material creation,
   better textures, a title or main menu.
7. Update `README.md` (its controls table says "to define" for the switch key and the progress list is out of
   date: switch, shooting, portal, teleport, rooms are done).
8. Week 6: Windows packaging (Packaging Settings, Development, Full Rebuild), see the course TP.

Planning rule: keep a 30 % safety margin, stay in whitebox until week 5, three good rooms beat eight rushed ones.
