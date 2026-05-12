# LockeBox — Visual System
**Version:** 1.1
**Studio:** Heavy C Development Studio, LLC

> This document defines the hard visual rules for LockeBox. Every rule here is a constraint, not a suggestion. AI-assisted code generation must follow these exactly. Deviations require explicit human approval and must be documented.

---

## 1. Design Philosophy

LockeBox should feel like an in-universe Trainer Toolkit: a high-density handheld command device for Nuzlocke players. The aesthetic reference points are physical hardware, tactical field readouts, cartridge contacts, scanline displays, status LEDs, and crisp Pokémon sprites inside framed viewfinders.

The tracker is a co-pilot, not a cockpit. The game screen is always the hero. The UI earns attention only when the player looks at it.

**Three words that define every design decision:** Dense. Tactical. Intentional.

---

## 2. Color System

All colors are defined as design tokens. No hardcoded hex values anywhere in component code — always reference tokens.

### 2.1 Background Scale

```dart
// lib/theme/app_colors.dart

// Backgrounds — obsidian/navy hardware, never pure black
static const bg0 = Color(0xFF060E20);  // App background — deepest layer
static const bg1 = Color(0xFF0B1326);  // Screen background, main surfaces
static const bg2 = Color(0xFF131B2E);  // Cards, panels, elevated surfaces
static const bg3 = Color(0xFF2D3449);  // Dividers, borders, subtle outlines
static const bg4 = Color(0xFF31394D);  // Hover/pressed states, inline inputs
```

### 2.2 Text Scale

```dart
static const textPrimary   = Color(0xFFDAE2FD); // Primary text — cool off-white
static const textSecondary = Color(0xFFD3C5AC); // Secondary, labels, captions
static const textMuted     = Color(0xFF7C8498); // Disabled, placeholders
static const textInverse   = Color(0xFF0B1326); // Text on accent backgrounds
```

### 2.3 Accent Colors

```dart
// Primary accent — electric amber. Used for primary actions and key warnings.
static const amber       = Color(0xFFFBBF24);
static const amberSoft   = Color(0xFFFFE1A7);
static const amberDim    = Color(0xFF795900); // Inactive/background amber
static const amberGlow   = Color(0x40FBBF24); // Glow overlay

// Secondary accent — cyan readout. Used for active tabs, scanners, viewfinders.
static const cyan        = Color(0xFF5DE6FF);
static const cyanDim     = Color(0xFF2FD9F4);
static const cyanGlow    = Color(0x405DE6FF);

// Hardware overlays
static const hardwareOutline = Color(0xFF4F4633);
static const rimHighlight    = Color(0x1ADAE2FD);
static const gridDot         = Color(0x14DAE2FD);
static const scanline        = Color(0x1A000000);

// Status semantics — used ONLY for their defined meaning
static const statusAlive   = Color(0xFF71FDBF); // Green — alive Pokémon, healthy HP
static const statusDanger  = Color(0xFFFFB4AB); // Red — deaths, fainted, critical HP
static const statusWarning = Color(0xFFE8A020); // Amber — at level cap, low PP, caution
static const statusMuted   = Color(0xFF626A7D); // Gray — missed encounter, dupes clause

// Interactive
static const interactive        = Color(0xFFFBBF24); // Tappable elements default
static const interactivePressed = Color(0xFF795900); // Pressed state
static const interactiveFocus   = Color(0x665DE6FF); // Focus ring
```

### 2.4 Pokémon Type Colors

Type colors are used for type chips on Pokémon cards, damage calc output, and move type indicators. These are the canonical values — never approximate.

```dart
static const typeColors = {
  'normal':   Color(0xFF9FA19F),
  'fire':     Color(0xFFE62829),
  'water':    Color(0xFF2980EF),
  'electric': Color(0xFFFAC000),
  'grass':    Color(0xFF3FA129),
  'ice':      Color(0xFF3DCEF3),
  'fighting': Color(0xFF9F2524),
  'poison':   Color(0xFF9141CB),
  'ground':   Color(0xFF915121),
  'flying':   Color(0xFF748FC9),
  'psychic':  Color(0xFFEF4179),
  'bug':      Color(0xFF91A119),
  'rock':     Color(0xFFAFA981),
  'ghost':    Color(0xFF704170),
  'dragon':   Color(0xFF5060E1),
  'dark':     Color(0xFF624D4E),
  'steel':    Color(0xFF60A1B8),
  'fairy':    Color(0xFFEF70EF),
};
```

### 2.5 Color Rules

**FORBIDDEN:**
- No pure `#000000` or `#FFFFFF` anywhere in the UI
- No off-token dark backgrounds. Use the obsidian/navy bg scale only.
- No decorative gradients in component code. Hardware glow/scanlines must come from tokens or shared Toolkit widgets.
- No ordinary drop shadows. Use border + background elevation; cyan/amber glow is reserved for active device states.
- No opacity-based color mixing for semantic colors — use the defined tokens

**REQUIRED:**
- Every background must use the bg scale in order (bg0 → bg1 → bg2 → bg3/bg4 for borders and hover)
- HP bars: green above 50%, amber 20–50%, red below 20%
- Death-related UI always uses `statusDanger`
- Alive Pokémon indicators always use `statusAlive`
- Level cap warnings always use `statusWarning`

---

## 3. Typography

### 3.1 Font Families

```dart
// Three fonts. Each has a specific role. Never swap them.

const fontDisplay = 'PressStart2P';   // Hero numbers and run identity only
const fontMono    = 'IBMPlexMono';    // All stats, numbers, code-like data
const fontBody    = 'IBMPlexSans';    // All prose, labels, descriptions
```

**Press Start 2P** — the pixel font. This is the soul of the hardware aesthetic. It is used ONLY for:
- Death count on the bottom sheet handle
- Current level cap on the bottom sheet handle  
- Run name on run history cards (large)
- The LockeBox wordmark / app title

**It is never used for:** body text, labels, descriptions, moves, route names, stat values, anything at small sizes. The restraint is what makes it land.

**IBM Plex Mono** — all numerical data. IVs, EVs, stats, damage ranges, PP counts, level numbers, met level, friendship value, experience. Anything that feels like a readout from a machine.

**IBM Plex Sans** — everything else. Route names, move descriptions, cause of death, nickname input, game names, settings labels, all body copy.

### 3.2 Type Scale

```dart
// Display — Press Start 2P only
// Note: Press Start 2P is naturally tall — keep line height tight (1.2) or text feels stretched.
static const displayLarge  = TextStyle(fontFamily: fontDisplay, fontSize: 24, height: 1.2);
static const displayMedium = TextStyle(fontFamily: fontDisplay, fontSize: 16, height: 1.2);
static const displaySmall  = TextStyle(fontFamily: fontDisplay, fontSize: 10, height: 1.2);
// MINIMUM size for Press Start 2P is 10px. Never render it smaller.

// Mono — IBM Plex Mono
static const monoLarge  = TextStyle(fontFamily: fontMono, fontSize: 16, fontWeight: FontWeight.w500);
static const monoMedium = TextStyle(fontFamily: fontMono, fontSize: 13, fontWeight: FontWeight.w400);
static const monoSmall  = TextStyle(fontFamily: fontMono, fontSize: 11, fontWeight: FontWeight.w400);
static const monoTiny   = TextStyle(fontFamily: fontMono, fontSize: 10, fontWeight: FontWeight.w400);

// Body — IBM Plex Sans
static const bodyLarge  = TextStyle(fontFamily: fontBody, fontSize: 15, fontWeight: FontWeight.w400);
static const bodyMedium = TextStyle(fontFamily: fontBody, fontSize: 13, fontWeight: FontWeight.w400);
static const bodySmall  = TextStyle(fontFamily: fontBody, fontSize: 11, fontWeight: FontWeight.w400);
static const labelLarge = TextStyle(fontFamily: fontBody, fontSize: 12, fontWeight: FontWeight.w600, letterSpacing: 0.8);
static const labelSmall = TextStyle(fontFamily: fontBody, fontSize: 10, fontWeight: FontWeight.w600, letterSpacing: 1.0);
```

### 3.3 Typography Rules

**FORBIDDEN:**
- Inter, Roboto, SF Pro, system fonts — never in the UI
- Press Start 2P below 10px
- Press Start 2P for anything that isn't a hero number or run identity
- Bold IBM Plex Mono above weight 600
- All-caps body text (labelLarge/labelSmall already have letter-spacing for caps use cases)

**REQUIRED:**
- All stat numbers (IVs, EVs, levels, damage) → `fontMono`
- All route names, move names, descriptions → `fontBody`
- Death count on handle → `displayMedium` in `statusDanger`
- Level cap on handle → `displaySmall` in `statusWarning`

---

## 4. Spacing & Shape

### 4.1 Spacing Scale

```dart
// 4px base unit. Everything is a multiple of 4.
static const s1  = 4.0;
static const s2  = 8.0;
static const s3  = 12.0;
static const s4  = 16.0;
static const s5  = 20.0;
static const s6  = 24.0;
static const s8  = 32.0;
static const s10 = 40.0;
static const s12 = 48.0;
static const s16 = 64.0;
```

### 4.2 Border Radius

```dart
// Hardware = sharp. Corners are small.
static const radiusNone   = BorderRadius.zero;                          // Emulator screen, image frames
static const radiusSharp  = BorderRadius.all(Radius.circular(2));      // Chips, tags, small elements
static const radiusSmall  = BorderRadius.all(Radius.circular(4));      // Cards, panels — the standard
static const radiusMedium = BorderRadius.all(Radius.circular(6));      // Buttons, inputs
static const radiusLarge  = BorderRadius.all(Radius.circular(8));      // Bottom sheet, modal
// MAXIMUM corner radius anywhere in the app is 8px.
// No pill shapes. No fully rounded cards. No circle avatars on Pokémon frames.
```

**FORBIDDEN:**
- `BorderRadius.circular()` values above 8
- Circular/pill buttons (use 6px radius instead)
- Fully rounded Pokémon sprite containers
- Any use of `StadiumBorder`

### 4.3 Borders & Elevation

No drop shadows. Elevation is communicated through:
1. Background color stepping (bg0 → bg1 → bg2)
2. 1px border using bg3 on elevated surfaces
3. The phosphor glow effect on active/selected states

```dart
// Standard card border
static const cardBorder = Border.all(color: bg3, width: 1);

// Active/selected state — amber glow border
static const activeBorder = Border.all(color: amber, width: 1);

// Phosphor glow — used on active tracker elements, selected Pokémon
// Implemented as a Container decoration with BoxShadow using amberGlow
static const phosphorGlow = BoxShadow(
  color: Color(0x33E8A020), // amberGlow
  blurRadius: 8,
  spreadRadius: 0,
);
// Use sparingly — max one glowing element visible at a time
```

### 4.4 Disabled States

Disabled controls use opacity, not color substitution:

```dart
// Disabled wrapper — applied to buttons, inputs, list items that are non-interactive
static const disabledOpacity = 0.4;

// Apply via Opacity widget or AnimatedOpacity (200ms) when state changes
// Disabled text: textMuted (already low-contrast)
// Disabled buttons: full opacity but background uses bg3 + textMuted text
```

### 4.5 Focus Rings (Keyboard Navigation — Windows / accessibility)

Windows users navigate with keyboard and need visible focus indicators.

```dart
// Focus ring — appears around any focusable widget when keyboard-focused
// 2px outline at amber, with 2px gap between widget and ring
static const focusRing = BoxDecoration(
  border: Border.fromBorderSide(BorderSide(color: amber, width: 2)),
  borderRadius: radiusSmall,
);

// Implementation: wrap interactive widgets with a Focus + AnimatedContainer
// that draws this ring when hasFocus == true.
// Touch interactions never show focus rings.
```

---

## 5. Component Specifications

### 5.1 Bottom Sheet Handle (Collapsed State)

The persistent tracker entry point. Always visible when game is running.

```
┌─────────────────────────────────────────────────┐
│  [☠ icon] [death count]   ⬆ LVL [cap]   [QA]    │
└─────────────────────────────────────────────────┘
```

(diagram uses placeholders — see iconography rules in §9 for actual icons)

- Background: `bg2` with top border `bg3`
- Height: 48px collapsed
- Death count: `displayMedium` (Press Start 2P, 16px) in `statusDanger` if deaths > 0, else `textMuted`
- Level cap: `displaySmall` (Press Start 2P, 10px) in `statusWarning` if enforcement on
- Quick action button: 32×32px, `bg3` background, 4px radius, amber icon
- Drag handle: 32×4px rounded pill centered at top, `bg4` color
- Touch target for expand: full width

**Quick action button states:**
- Default: "Mark route as encountered" (when standing on a claimable route — if known)
- Fallback: "Log a death" (skull icon)
- Never changes label mid-session — tap opens the action sheet, not a specific action

### 5.2 Pokémon Team Card

The most-viewed surface in the app. Two states: collapsed and expanded.

**Collapsed (default):**
```
┌──────────────────────────────────────────────┐
│ [sprite 40px] [NICKNAME]  Lv.[##]  [  HP   ]│
│               [Type][Type]         [item]    │
└──────────────────────────────────────────────┘
```

- Container: `bg2` background, 1px `bg3` border, 4px radius
- Sprite: 40×40px, pixel-perfect (nearest-neighbor filtering), no container border
- Nickname: `bodyLarge` `textPrimary`
- Level: `monoMedium` `textSecondary`, "Lv." in `textMuted`
- HP bar: full width below nickname row. Green/amber/red thresholds per §2.5
- Type chips: use the standard `TypeChip` widget — `radiusSharp` (2px), full type color background, `bodySmall` weight 600, white text, all-caps
- Item: small icon + `bodySmall` `textSecondary`, right-aligned
- Shiny indicator: small ✦ in `statusWarning` next to nickname if shiny

**Expanded (tap to flip):**
- Reveals: full stat block, IV bars, EV bars, nature, ability, moves + PP
- Stats/IVs/EVs: `monoSmall` for labels, `monoMedium` for values
- IV display: inline bar (0–31), color-coded per §2.5 rules
- EV display: progress bar toward 252, total shown as `monoTiny` "234 / 510"
- Nature: `bodySmall` with stat modifiers colored (amber for boosted, red for reduced)
- Moves: listed with type chip, PP as `monoSmall` current/max
- Transition: 150ms fade, no slide animation

**Dead (in graveyard):**
- Same card layout but `bg1` background (desaturated)
- Sprite desaturated (use ColorFilter.matrix)
- Cause of death: `bodySmall` `statusDanger` below nickname
- Level at death in `monoMedium` `textSecondary`
- No HP bar. No type chips. No expand gesture.

### 5.3 Route Row

```
┌──────────────────────────────────────────────────┐
│ ● [Route Name]          [Species caught]  [Status]│
└──────────────────────────────────────────────────┘
```

- Status indicator dot (8px): amber = unclaimed, green = caught, red = missed, muted = dupes/skip
- Route name: `bodyMedium` `textPrimary`
- Species caught: sprite (24px) + `bodySmall` `textSecondary` nickname
- Status badge: `labelSmall` all-caps, colored text, no background
- Tap: opens encounter logging sheet or shows caught Pokémon detail

### 5.4 Badge Display

Individual badge cell — 6 across for most regions.

- Unclaimed: `bg3` background, placeholder silhouette in `textMuted`, 4px radius
- Claimed: badge art, amber glow border (`activeBorder` + `phosphorGlow`)
- Size: 44×44px cells with `s2` gap
- No labels under badges — tooltip on long press only

### 5.5 IV / EV Bar

```
ATK  ████████████░░░░  29
```

- Label: `monoTiny` `textSecondary`, fixed 3-char width (left-aligned)
- Bar: thin (4px height), `bg4` background, fill color = value-based
  - IV 0–9: `statusDanger`
  - IV 10–19: `statusWarning`
  - IV 20–29: `textSecondary`
  - IV 30–31: `statusAlive` (with phosphor glow on 31)
- Value: `monoSmall` right-aligned, `textPrimary` for perfect IVs, `textSecondary` otherwise
- EV bar: same structure, fill is always `amber`, fill width = value/252

### 5.6 Buttons

```dart
// Primary action button
// Background: amber, text: textInverse (dark), fontMono medium, 6px radius, 44px min height
// NEVER use for destructive actions

// Secondary button  
// Background: bg3, text: textPrimary, border: bg3, 6px radius
// Used for cancel, neutral actions

// Destructive button
// Background: bg2, text: statusDanger, border: statusDanger (1px), 6px radius
// Used for: confirm death, delete run

// Icon button
// Background: bg3, icon: textSecondary, 32×32px, 4px radius
// Active state: bg4 background, amber icon
```

**FORBIDDEN button patterns:**
- Pill/rounded buttons
- Gradient backgrounds on buttons
- Drop shadow on buttons
- Full-width primary buttons on desktop/tablet (max 320px wide, centered)

### 5.7 Damage Calc Output

```
[Move Name]  [TYPE]
────────────────────────────────
Min  ████████░░░░░░  48 (32%)
Max  ████████████░░  72 (48%)
                    2HKO
```

- Move name: `bodyMedium` `textPrimary`
- Type chip: standard type chip
- Separator: 1px `bg3` line
- Bar labels: `monoSmall` `textMuted`
- Bars: 6px height, fill = `amber` for dealt / `statusDanger` for taken
- Values: `monoMedium` right-aligned
- OHKO/2HKO label: `labelSmall` in `statusDanger` / `statusWarning` / `textSecondary`

---

## 6. Sprite Rendering

**All Pokémon sprites are rendered pixel-perfect.** This is non-negotiable.

```dart
// Always use FilterQuality.none for sprites
// Never bilinear/trilinear filter a sprite
Image(
  image: cachedNetworkImage,
  filterQuality: FilterQuality.none,
  width: targetSize,
  height: targetSize,
  fit: BoxFit.contain,
)
```

**Sprite sizes by context:**
- Team card collapsed: 40×40px
- Team card expanded: 64×64px
- Route row caught indicator: 24×24px
- Graveyard card: 40×40px (desaturated: `ColorFilter.matrix(greyMatrix)`)
- Line builder calc matrix: 32×32px
- Share card export: 96×96px

**Shiny sprites:** Displayed when `isShiny == true`. Uses the shiny sprite URL from PokeAPI. A `✦` indicator is always shown alongside — never rely on the sprite alone to indicate shininess (some shiny sprites are subtle).

---

## 7. Emulator Screen Frame

The game screen has specific framing rules depending on bezel state.

**No bezel (default free tier):**
- Screen renders edge-to-edge in its allocated area
- 1px `bg3` border around screen area
- No corner radius on the screen itself (`radiusNone`)

**With bezel (Supporter Pack):**
- Bezel image rendered as container around screen texture
- Screen positioned to match the real hardware screen cutout
- Pixel-perfect bezel art — never stretched
- Bezel colors are cosmetic data, not hardcoded

**Fast forward indicator:**
- Small amber badge `[2×]` `[4×]` etc. in top-right of screen area
- `displaySmall` Press Start 2P, `amber` color, `bg1` background at 80% opacity
- Only visible while fast forward is active

---

## 8. Motion & Animation

**The app is not animated for delight. It is animated for clarity.**

```
Standard transition:    150ms, ease-out
Sheet expand/collapse:  250ms, ease-in-out
Card flip (stats):      150ms, fade only (no 3D flip)
Death event:            Screen flash: statusDanger at 30% overlay, 100ms fade out
HP bar fill:            Instant on sync, no animation
Route status change:    150ms color transition on dot
Badge earn:             200ms scale 1.0 → 1.15 → 1.0 + phosphor glow pulse (one-shot)
```

**FORBIDDEN animations:**
- Bounce/spring physics on any UI element
- Continuous/looping animations except supported team card backgrounds (Supporter Pack only)
- Slide animations exceeding 250ms
- Any animation that plays while the emulator is running at full speed
- Hero transitions between routes
- Parallax effects

---

## 9. Iconography

Use **Material Symbols** (outlined weight) as the base icon set. Do not mix icon families.

Key icon assignments (do not change these — consistency matters):
```
Death / fainted:    skull (or custom pixel skull asset)
Route claimed:      catching_pokemon (or pokeball custom)
Route missed:       close
Level cap:          arrow_upward  
Sync from save:     sync
Fast forward:       fast_forward
Save state:         save
Run reset state:    flag
Graveyard:          cemetery (or custom)
Settings:           settings
Badge earned:       star (filled)
IV perfect (31):    diamond (filled, amber)
Shiny:              auto_awesome (amber)
```

Custom pixel-art icons for skull and pokéball are preferred over Material symbols for these two specific uses. All others use Material Symbols outlined.

---

## 10. The Forbidden List

A complete list of things that must never appear in LockeBox UI:

| Forbidden | Why |
|---|---|
| Pure `#000000` or `#FFFFFF` | Too stark, kills the warm hardware feel |
| Blue-tinted dark backgrounds | Wrong temperature for the aesthetic |
| Drop shadows | Use elevation + border instead |
| Gradients (except phosphor glow) | Cheap, fights the hardware vibe |
| Border radius > 8px | Softness is wrong for this aesthetic |
| Pill/stadium shaped buttons | Same reason |
| Inter, Roboto, SF Pro | Generic, kills identity |
| Press Start 2P below 10px | Illegible and misused |
| Press Start 2P for body text | Destroys the impact of its special uses |
| Bilinear sprite filtering | Blurs pixel art — always nearest-neighbor |
| Animations > 250ms | Sluggish, fights the snappy hardware feel |
| Bounce/spring physics | Wrong material for a hardware aesthetic |
| Circle avatar frames for Pokémon | Pokémon sprites aren't avatars |
| Full-width buttons on desktop | Looks mobile-ported, not designed |
| Any element with `elevation:` > 0 in Material theme | Use bg scale instead |
| `Colors.white` in any widget | Use `textPrimary` token |
| `Colors.black` in any widget | Use `bg0` token (or `textInverse` if it's text on amber) |
| Hardcoded hex colors outside `app_colors.dart` | Token violations break the system |

---

*This document is the design source of truth. When in doubt, ask: does this look like it belongs on a Game Boy Advance? If the answer is no, change it.*
