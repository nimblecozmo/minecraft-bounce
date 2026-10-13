# Minecraft Bounce — Game Specification

## Overview
A single-file HTML browser game (vanilla JS, no dependencies). Player dodges
Minecraft mobs and trees by jumping. Hit detection = death animation + retry
screen. Three screens: Title → Game → Death. Main Menu button returns to
Title.

## Visual Style
- **Minecraft-y aesthetic**: 8-bit pixel art, blocky shapes, grass blocks,
  oak/birch trees, common mobs.
- **Font**: Use a Minecraft-style font (Google Font: "Press Start 2P" or
  pixel-style monospace fallback). Logo "MINECRAFT BOUNCE" rendered in
  chunky pixelated style.
- **Color palette** (authentic MC):
  - Sky: #4A90E2 (daytime blue) gradient
  - Grass: #5D8C3A top, #8B6F3F dirt below
  - Steve: #C28E5C skin, #4A8AC7 shirt, #2E4F8B pants
  - Alex: #C28E5C skin, #6EBA56 shirt, #6E4A2E pants, longer hair
  - Tree trunks: #6E4A2E (oak) / #DCDCC0 (birch)
  - Leaves: #3D8B2C (oak) / #5DAB3D (birch)
  - Mobs: zombie (#5BA85B), skeleton (#DCDCC0), creeper (#5DAB3D)
- **Title screen background**: blocky grass + sky + trees in distance, with
  a white-ish translucent overlay (rgba(255,255,255,0.55)) so black text
  is readable.

## Canvas
- 900×620 fixed size
- Centered in viewport (flex layout, dark backdrop)
- Pixel art rendered via small rects/canvas drawing primitives — not images.
  This keeps the file self-contained (single HTML, no external assets).

## Screens

### Screen 1: Title
- Background: minecraft scene (sky, hills, trees) drawn on canvas
- White-ish overlay (0.55 alpha) so text is readable
- Logo "MINECRAFT BOUNCE" centered, top — pixelated, ~64px tall, with
  subtle drop shadow
- Subtitle: "A dodging adventure"
- Two buttons stacked vertically, centered:
  1. **CHARACTER SELECTOR** → goes to Screen 1b
  2. **PLAY** → only enabled if a character has been selected
     (disabled-state visually: grayed out, unclickable)

### Screen 1b: Character Selector
- Same background + overlay
- Title: "CHOOSE YOUR CHARACTER"
- Two large character cards side-by-side:
  - **STEVE** (drawn pixel-art on a 128×192 canvas each)
  - **ALEX** (drawn pixel-art on a 128×192 canvas each)
- Each card: name, pixel art, brief tag ("Classic" / "Adventurer")
- Hover: slight scale up
- Click: select character (highlighted border), return to Title screen
  with the chosen character now shown small next to PLAY button

### Screen 2: Game (Side-scroller, auto-scroll right)
- Sky + grass ground, parallax (clouds drift slow, ground scrolls)
- Player character (selected Steve or Alex) on left side of screen
- World scrolls right-to-left (player appears to run forward)
- Obstacles spawn from right edge, scroll left:
  - **Trees**: oak OR birch (50/50). Tall block: trunk + leaf canopy.
  - **Mobs**: zombie / skeleton / creeper. Blocky 2-leg silhouette.
- **Controls**:
  - `Space` (or `↑` arrow, or `W`): jump
  - Tap space while on ground → single jump
  - Hold space longer → higher jump (charge mechanic, optional but fun)
  - Tap again in air → double jump (one extra mid-air jump)
- **Physics** (tuned for relaxed pacing):
  - Gravity: 0.5 px/frame²
  - Jump velocity: -12 px/frame
  - Double-jump velocity: -10 px/frame
  - Ground y: 480 (px from top of canvas)
  - Player hitbox: 32×64 px
  - Scroll speed: 2.5 px/frame base, max 1.4× (gradual ramp)
  - Initial spawn delay: 90 frames (1.5s) before first obstacle
  - Min gap between obstacles: 450 px
- **Collision**:
  - AABB hitbox check against each obstacle
  - On hit → Screen 3
- **Scoring**:
  - +1 per obstacle cleared (off-screen left)
  - Speed increases every 10 obstacles (cap at 1.5× starting speed)
  - Display score top-right in pixel font

### Screen 3: Death
- Same background, darker overlay
- **Death animation** (1.5 seconds):
  - Player falls over to the side (rotate 90°)
  - White "death particles" emit upward (small white squares float up
    and fade out — like Minecraft's death particles)
- **Death screen buttons appear IMMEDIATELY** on death (don't wait for
  animation) so the user has instant access to Play Again / Main Menu
- After animation completes:
  - Big text: "YOU DIED!" in red, pixel font
  - "Score: N"
  - Two buttons (already visible since death started):
    - **PLAY AGAIN** → restart Screen 2 with same character
    - **MAIN MENU** → go to Screen 1 (Title)

## Game States
```
TITLE → CHARACTER_SELECTOR → TITLE → GAME → DEATH
                                        ↑         ↓
                                        └──PLAY AGAIN
                                        MAIN MENU↓ (from DEATH)
```
State managed via a single `state` variable: `'title' | 'select' | 'game' | 'death'`.

## Obstacle Spawning
- Spawn at right edge, x = 900
- Min gap between obstacles: 400 px (so player can recover)
- Tree:Mob ratio: 1:1
- Each tree is randomly oak or birch
- Each mob is randomly zombie/skeleton/creeper
- Obstacles removed when x < -100

## Audio (optional)
- No audio in v1. Could add simple WebAudio synth for jump/hit later.

## File Structure
- Single file: `/home/blake/dev/minecraft-bounce/index.html`
- All CSS in `<style>` tag
- All JS in `<script>` tag at end of `<body>`
- Google Fonts loaded via `<link>` (Press Start 2P or VT323)

## Implementation Notes
- Use `requestAnimationFrame` for game loop
- Use canvas `ctx.fillRect` for all pixel art (no `<img>` deps)
- Pre-draw character sprites once, blit to canvas each frame
- Pre-draw obstacle sprites once, blit per frame
- Hitbox slightly smaller than visual sprite (forgiving)
- Press Start 2P from Google Fonts, fallback to monospace

## Acceptance Criteria
- [x] File opens directly in browser, no server needed
- [x] Title screen with logo, overlay, two buttons
- [x] Character selector with Steve and Alex drawn pixel-art
- [x] Selecting a character returns to title, PLAY button now enabled
- [x] Pressing PLAY starts the game
- [x] Player can jump with Space
- [x] Trees and mobs spawn and scroll left
- [x] Hitting obstacle triggers death animation
- [x] Death animation: player tilts + white particles
- [x] Death screen shows score + Play Again / Main Menu
- [x] Play Again restarts with same character
- [x] Main Menu returns to title screen
- [x] Score increments per obstacle cleared
- [x] Speed gradually increases
- [x] All within a single HTML file
- [x] No external image dependencies (all pixel art via canvas)

## Easter Egg: Tool Master Mode

**Trigger**: Click on the **"M"** in the "MINECRAFT" logo on the title screen.
A pulsing gold starburst appears next to the M and an "EASTER EGG ACTIVE"
label is drawn below the logo.

**Effect in-game** (when active):
- The player auto-equips tools to defeat obstacles instead of dying:
  - **Diamond sword** (cyan blade) when near **mobs** (zombie / skeleton).
    Mobs get sliced and play the Minecraft death animation (rotate 90°
    + **15 white particles** floating upward, same style as the
    character death). Player takes no damage.
  - **Iron axe** (gray head + brown handle) when near **trees** (oak /
    birch). The tree rotates from 0° to 90° (falls over), then drops
    **2 wood blocks** (brown squares with darker top/left edge). Wood
    blocks are auto-picked up by the player when within 8px, which
    spawns a burst of **6 gold/brown sparkle particles** and increments
    the **wood counter** in the HUD.
  - **Flint and steel** (gray flint + gold steel) when near a
    **creeper**. The creeper explodes in a layered orange/yellow
    blast (radius grows for 30 frames, then fades), plus **20 orange
    / yellow firework-style particles** spreading in a circle, plus
    **8 gray smoke particles** rising upward.
- The death-collision check is bypassed for obstacles that are
  consumed by the easter egg.
- The tools are drawn in the player's right hand with a small swing
  animation when they strike an obstacle.
- Click the M again to toggle the easter egg off.
- Score: +1 for mobs, +2 for trees, +3 for creepers.
- HUD shows "EASTER EGG / TOOL MASTER ON" in top-left with pulsing
  gold border, plus a wood block counter ("🟫 x N") once wood is
  collected.

## Easter Egg 2: UNO REVERSE!

**Trigger**: When the easter egg is on AND a skeleton is between 80-180px
from the player (close but not touching).

**Sequence** (~3 seconds total):
1. The world **freezes** (no clouds, no scroll, no obstacle movement).
2. The player pulls out a **UNO reverse card** (yellow with red border).
3. The skeleton **turns around** (flipped horizontally, facing away).
4. The skeleton starts **shooting itself** with gold spark particles
   every 5 frames for 180 frames (3 sec at 60fps).
5. A "?!?!" speech bubble appears above the skeleton's head.
6. A big **"UNO REVERSE!"** text appears in the top-center of the
   screen (red with yellow stroke, bouncing in for 20 frames).
7. After 3 seconds, the skeleton dies the Minecraft death animation
   (rotate 90° + 15 white particles floating upward, same style as
   the character death) and is removed.
8. Score: +2 bonus (skeletons are harder to defeat).
9. The world unfreezes and gameplay resumes.

While the UNO effect is active, the skeleton cannot damage the player
(collision detection ignores `_unoFlipped` skeletons).

## Settings

The settings menu (gear icon, top-right of title screen) is **empty**
in this build. The "Blood and Gore" toggle has been removed — this
is the **no-blood version** of the game:

- **No blood particles** anywhere — neither from easter-egg kills
  nor from character death
- **Mob death effect**: mobs fall over (rotate 90° like the
  character death) and spawn **15 white particles** floating
  upward — the exact same animation style as the character death
- **No bonepiles** when skeletons are killed
- **No floor holes** when creepers are killed
- **Character death** spawns a **gray smoke puff** (30 small gray
  particles) instead of a blood explosion
- **Nuke-killed mobs** also use white particles (10 each) instead
  of red blood
- The setting menu simply shows "No settings yet" (placeholder
  text, ready for future options)

This is a clean, family-friendly version of the game with no
gore at all.

## Easter Egg 4: Structure Build (every 50 score points)

**Trigger**: When the player's score crosses a 50-point milestone
(50, 100, 150, ..., 700), the world freezes and a **random unbuilt**
structure is built. Each structure only builds once per game so the
player sees different structures in different runs.

**Random selection:**
- All 14 structures are eligible from the start
- When a milestone is crossed, one is picked at random from the
  remaining unbuilt ones
- It's added to the `builtStructures` set so it won't be picked again
- This continues until all 14 are built (after the 700 milestone)

**The build sequence:**

1. **Walk to base** — Player walks from current position to the left
   side of the structure's footprint.
2. **Build blocks** — Player walks to each block position in
   bottom-up, left-to-right order, placing a block at each position.
   As the structure rises, the player's y-position rises too (climbs
   up to 32px above ground) so they appear to climb the structure.
3. **Walk to entry** — Once all blocks are placed, the entry door
   swings open (rotates 90° outward).
4. **Walk through** — Player walks through the interior, past the
   center, to a position just left of the exit door.
5. **Exit** — The exit door swings open, and the player walks past
   the structure to the right.
6. **Resume** — The world unfreezes, all obstacles are cleared, the
   player is reset to the start position, and gameplay continues.

**The structures (14 total, every 50 score points):**

| Score | Structure | Notes |
|-------|-----------|-------|
| 50 | Small Oak House | 6x6 — alternating oak plank walls, dark roof |
| 100 | Cobblestone Tower | 5x7 — solid base + 4 wall levels + 3 crenellations |
| 150 | Village Church | 5x8 — white walls + red roof + steeple |
| 200 | Viking Longhouse | 7x5 — long oak walls with window slits, shingle roof |
| 250 | Igloo | 5x4 — snow walls with ice windows, dome top |
| 300 | Jungle Temple | 5x5 — mossy cobblestone walls, mossy slab roof |
| 350 | Desert Pyramid | 5x5 — sandstone walls, stepped pyramid, capstone |
| 400 | Witch Hut | 3x4 — small oak walls, red+white mushroom roof |
| 450 | Stronghold Portal Room | 6x4 — stone brick + active purple End Portal |
| 500 | Ancient City Portal Frame | 5x6 — reinforced deepslate + green portal glow |
| 550 | Ocean Monument | 5x6 — scaled prismarine + gold guardian statues |
| 600 | Nether Fortress | 6x5 — nether brick + orange blaze spawner in roof |
| 650 | End City | 5x6 — purpur walls + white shulker on top |
| 700 | Woodland Mansion | 5x5 — dark oak walls (just one wing) |

**Scaling rule (per user spec):**
- No structure is taller than 7 blocks (98px) — same as a tree
- Big structures (ocean monument, ancient city, end city) are scaled
  down to a representative portion (e.g. 1 portal, 2 prisms, 1 wing)
- Multi-room structures use the most iconic room (stronghold → portal
  room, ancient city → big portal frame)

**Visual design:**
- 14px per block (BLOCK constant)
- Each block has a base color + 2px dark shadow (bottom/right) + 2px
  light highlight (top/left) for a 3D pixel-art look
- Doors are 1 block wide × 2 blocks tall, brown with a gold handle
- Structure name displayed above the structure during build

**Files:**
- `/home/blake/dev/minecraft-bounce/index.html` — 2383 lines, 95.4 KB
- `/home/blake/dev/minecraft-bounce/spec.md` — updated with structure list

## Settings Menu

**Trigger**: A **"⚙ Settings"** button appears on the title screen,
below the **CHARACTER SELECTOR** and **PLAY** buttons. The button
uses the same blue background (`#4A8AC7`) and same Press Start 2P
font as the other title buttons — it's just labeled with the gear
icon ⚙ to make it visually distinct.

**Visual design:**
- Button: blue (`#4A8AC7`) with the standard Minecraft button styling
  (4px black border, 6px black drop shadow, hover-lighten effect)
- Text: "⚙ Settings" in Press Start 2P, uppercase
- Position: **top-right corner** of the title screen (16px from top
  and right edges, `position: absolute` relative to the game stage)
- The button uses a slightly smaller font (14px) and padding so it
  doesn't dominate the title screen

**Title-only visibility:**
- The settings button is **only visible on the title screen**
- A new `updateSettingsButton()` function shows/hides the button
  based on the current state (visible when `state === 'title'`,
  hidden during `select`, `game`, and `death`)
- Called from every state transition (Character Selector, Play,
  Play Again, Main Menu, nuke, etc.)
- **The button is in the static HTML** so it's visible on the
  initial page load (no need to play a round first to see it)

**Settings menu overlay:**
- Full-screen dark dim background (`rgba(0,0,0,0.7)`) over the title
- Centered dark gray panel (`#2A2A2A`) with the same Minecraft border
  styling as the buttons
- "SETTINGS" title in Press Start 2P, white, large
- Settings body — currently shows "No settings yet" placeholder
- "EXIT" button at the bottom — same blue styling as title buttons
- Closes the menu and returns to the title screen

**Files:**
- `/home/blake/dev/minecraft-bounce/index.html` — 2383 lines, 95.4 KB

## Easter Egg 3: NUKE

**Trigger**: A red **"☢ NUKE"** button appears at the top-center of the
screen during gameplay (only when the easter egg is on).

**Click the button** to send a nuke down:

**Phase 1 — Falling** (~0.5 sec, 30 frames):
- A dark nuke with a yellow radiation trefoil falls from the top of
  the screen toward the center of the ground.
- A gray smoke trail follows behind it.
- A pulsing yellow glow surrounds the nuke.
- The world is frozen (no scroll, no spawns, no movement).

**Phase 2 — Explosion** (one frame):
- All obstacles on screen are instantly killed: every tree falls, every
  mob/creeper does its Minecraft death animation. They all award +2 score.
- 80 explosion particles are spawned from the impact point.

**Phase 3 — Orange screen for 1 second** (post-boom frames 0-60):
- The whole screen is tinted bright orange (60% alpha) — held for 1 second.
- A big "NUKE!" text in red with yellow stroke (bouncing with screen shake).
- Expanding white shockwave ring grows from r=30 to r=510.
- A bright white flash on the very first frame.

**Phase 4 — Particles fall through the floor** (post-boom frames 60-420, 6 sec):
- The orange tint ends and the **title screen is revealed underneath**.
- Each of the 80 particles has a unique `fadeStart` between 0-320 frames
  (within the 360-frame fall-and-fade window).
- Each particle, once its `fadeStart` is reached, fades out over 30 frames.
- Particles fall through the floor under strong gravity (vy += 0.35 per frame).
- No air drag — particles keep falling until they fade.
- All particles are gone by frame 420 (6 seconds after the orange phase ends).
- The world stays frozen throughout the whole post-boom sequence.

**Phase 5 — End / Play again** (post-boom frame 420+):
- State switches to `'title'`, so the title screen draws as the main scene.
- The game world is reset (`resetGame()`).
- Character Selector and Play buttons are restored.
- Easter egg remains on, so the player can Nuke again next round.
- NUKE button is hidden (only shows during gameplay).
