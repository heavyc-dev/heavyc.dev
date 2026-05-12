# LockeBox — Project Specification
**Version:** 0.5.1-spec
**Studio:** Heavy C Development Studio, LLC  
**Last Updated:** 2026-05-11

---

## Table of Contents

1. [Overview](#1-overview)
2. [Technical Stack](#2-technical-stack)
3. [Licensing & Open Source Strategy](#3-licensing--open-source-strategy)
4. [Monetization](#4-monetization)
5. [Pokémon Data & Sprites](#5-pokémon-data--sprites)
6. [Phased Roadmap](#6-phased-roadmap)
7. [Feature Specification — Emulator Core](#7-feature-specification--emulator-core)
8. [Feature Specification — Nuzlocke Tracker](#8-feature-specification--nuzlocke-tracker)
9. [Feature Specification — Line Builder](#9-feature-specification--line-builder)
10. [Feature Specification — Widgets](#10-feature-specification--widgets)
11. [Post-Launch Features](#11-post-launch-features)
12. [Game Data](#12-game-data)
13. [Save File Parsing — Technical Reference](#13-save-file-parsing--technical-reference)
14. [UI / UX Principles](#14-ui--ux-principles)
15. [Out of Scope](#15-out-of-scope)
16. [Open Questions](#16-open-questions)

---

## 1. Overview

LockeBox is a cross-platform GBA and DS emulator purpose-built for Nuzlocke runs. It is not a general emulator with a tracker bolted on — the tracker is the product. Every emulation feature exists in service of the Nuzlocke experience.

**Implementation status:** MVP development is in progress. For the current repo/file ownership map, implemented session behavior, and known gaps between target docs and code, see [CURRENT_STATE.md](./CURRENT_STATE.md).

**Target platforms:** Windows, iOS, Android  
**Target audience:** Nuzlocke players, challenge runners, content creators, hardcore Pokémon fans  
**Core value proposition:** The only emulator that understands the run you're doing  
**Business model:** Free and open source with optional one-time cosmetic IAP ("Support the dev")

---

## 2. Technical Stack

### Framework
**Flutter (Dart)**  
Rationale: True cross-platform native compilation for all three targets, excellent FFI for C/C++ emulation cores, strong rendering pipeline (Impeller), fast iteration. No web target needed.

### Emulation Cores

#### GBA — mGBA
- **Repo:** https://github.com/mgba-emu/mgba
- **License:** MPL 2.0 — file-level copyleft only. Permissive for app integration.
- **Integration:** Compiled as a static/shared library per platform, bound via `dart:ffi`
- **Renderer:** mGBA software renderer → raw RGBA framebuffer → Flutter `Texture` widget via `dart:ui` GPU upload
- **BIOS:** HLE BIOS by default. User may provide their own for better accuracy.
- **Audio:** mGBA audio buffer → platform audio via native channel

#### DS — melonDS
- **Repo:** https://github.com/melonDS-emu/melonDS
- **License:** GPL 3.0 — see §3 for full strategy
- **Integration:** Same FFI pattern as mGBA — compiled per platform, framebuffer → Flutter Texture
- **BIOS/Firmware:** Required from user. App prompts on first DS ROM load and stores in sandbox. Never shipped with the app. melonDS includes an open HLE BIOS as fallback.
- **Dual screen:** Two Texture widgets, configurable layout (stacked, side-by-side, swappable, primary focus)
- **iOS:** Fully viable on App Store. Delta ships melonDS on the App Store with full DS support. The DS uses ARM processors — same architecture as iPhone — so the interpreter penalty without JIT is minimal. No JIT workaround needed.

#### GB/GBC — SameBoy (Phase 3)
- **Repo:** https://github.com/LIJI32/SameBoy
- **License:** MIT — fully permissive
- **Enables:** Gen 1 (Red/Blue/Yellow) and Gen 2 (Gold/Silver/Crystal)

### DS Core Licensing Landscape
Every major open source DS emulator (melonDS, DeSmuME, NooDS) is GPL licensed. There is no MPL or MIT DS core available. This makes the open source app strategy in §3 the correct path — it resolves all GPL friction cleanly.

### State Management
**Riverpod** — reactive, well-suited for tracker state that responds to emulator events.

### Storage
- **Local:** SQLite via `drift` — all run data, encounter tables, game metadata, cached Pokémon data
- **Backup:** iCloud (iOS) and Google Drive (Android/Windows) — optional, opt-in, no account required
- **ROM storage:** User-supplied. Stored in app sandbox or picked via file picker. No ROM distribution.

### Platform Matrix

| Feature | Windows | iOS | Android |
|---|---|---|---|
| GBA (mGBA) | ✅ | ✅ | ✅ |
| DS (melonDS) | ✅ | ✅ | ✅ |
| GB/GBC (SameBoy) | ✅ Phase 3 | ✅ Phase 3 | ✅ Phase 3 |
| Controller | XInput/DInput | MFi + Bluetooth | Android gamepad API |
| File Picker | Full filesystem | Files app sandbox | SAF |
| iCloud backup | ❌ | ✅ | ❌ |
| Google Drive backup | ✅ | ✅ | ✅ |
| Home screen widget | ✅ | ✅ | ✅ |

---

## 3. Licensing & Open Source Strategy

### Decision: LockeBox is Open Source (GPL 3.0)

**Why:**

**Legal clarity.** melonDS is GPL 3.0. GPL copyleft requires derivative works to also be GPL 3.0. Rather than architecting around this (subprocess isolation, seeking a license exception), the cleanest path is to open source the whole app. This is the same decision Delta made — the gold standard DS emulator on the App Store — and it works.

**App Store compatibility.** GPL 3.0's anti-tivoization clause technically conflicts with App Store signing requirements. In practice, Apple has not enforced this against open source emulators and Delta ships without issue. Publishing source on GitHub satisfies the spirit of the license as the community interprets it.

**Business model compatibility.** GPL explicitly permits charging for software and running paid IAP. The Supporter Pack (§4) is fully compatible. You are selling a cosmetic experience, not a software license.

**Community trust and leverage.** The Nuzlocke audience is technical and community-oriented. Open source builds credibility. Contributors may submit game data files, ROM hack support, and bug fixes — free labor with attribution.

**Precedent.** Delta Emulator: open source on GitHub, IAP cosmetics on App Store, thriving. Exactly the model.

### License Summary by Component

| Component | License | Obligation |
|---|---|---|
| mGBA | MPL 2.0 | Publish changes to mGBA files only |
| melonDS | GPL 3.0 | App source must be GPL 3.0 |
| SameBoy | MIT | Copyright notice in credits |
| LockeBox app | GPL 3.0 | Full source on GitHub |
| Game data JSON | GPL 3.0 | Included in repo |
| Supporter Pack art assets | TBD — see open questions | Consult attorney before launch |

### Attribution Requirements
App must include a licenses screen crediting: mGBA (Jeffrey Pfau), melonDS (Arisotura and contributors), SameBoy (Lior Halphon), PokeAPI, and all Dart/Flutter packages.

---

## 4. Monetization

### Free Tier — Everything Functional
All emulation and tracker features are free forever. No paywalled functionality, no ads, no telemetry, no account required.

### One-Time Upgrade — "Supporter Pack"
Single IAP, ~$4.99. Framed as "support the developer."

| Category | Items |
|---|---|
| Emulator bezels | GBA SP (Onyx, Flame Red, Pearl Pink), DS Lite, Game Boy Pocket, Heavy C custom skin |
| UI themes | 4–6 tracker color schemes (Blood Red "Hardcore", Seafoam "Chill", Midnight "Focus", etc.) |
| Badge case styles | Alternate art styles |
| Graveyard styles | Gothic stone, pixel cross, minimal plaque, memorial garden |
| Widget themes | 3 visual styles |
| Run export card templates | 3 additional share card layouts |
| Animated team cards | Subtle background animations |

**Principle:** A player should never feel like they're missing something functional. They should occasionally see a cosmetic they want and feel good supporting the project.

---

## 5. Pokémon Data & Sprites

> **See [DATA_ARCHITECTURE.md](./DATA_ARCHITECTURE.md) for the complete data layer design.** This section is the high-level summary; the architecture doc has the full detail on per-game databases, ROM identification, sync, and contribution workflow.

### Source of Truth: GitHub-hosted Data Repository

LockeBox does not rely on PokeAPI for game data. PokeAPI provides canonical retail data, but ROM hacks change base stats, types, abilities, learnsets, moves, type charts, trainer teams — and crucially, hacks like Radical Red and Crystal Clear *add* brand-new content (Pokémon from later generations, new routes, new moves) that has no canonical equivalent to override.

LockeBox maintains a public GitHub repo (`heavyc-studio/lockebox-data`, GPL 3.0) with **per-game complete databases**: each retail game and each ROM hack ships its own complete dataset. No inheritance, no merging, no override resolution.

Each game folder contains:
- Complete Pokémon, move, ability, item, and type chart data for that game
- Routes, encounter tables, gym leader/E4/Champion teams
- Level caps, badge metadata
- Custom sprites for hack-native species (when applicable)

### ROM Identification by Hash

When a user loads a ROM, the app computes its SHA-1 hash and looks up the corresponding `gameId` in a registry (`rom_hashes.json`). This unambiguously identifies retail games AND specific ROM hack versions. ROM headers alone can't distinguish a Radical Red ROM from retail FireRed (both report `BPRE`); hashing solves that.

If the hash isn't recognized, the app prompts the user to check GitHub for support; if the hash is added later (community PR), the user gets prompted again on the next ROM load.

### On-Demand Download

The app does NOT eagerly download data for every supported game. When a user loads a ROM:
- Hash → gameId → check local cache
- If cached, ready instantly
- If not, download just that game's data folder (~200-400 KB gzipped)
- Cache locally; never re-download unless data updates

A user who only plays Emerald never downloads Radical Red's data.

### Bundled Fallback

The app ships with snapshot data for the MVP retail games (Emerald, FireRed) so first launch works without network. ROM hack data is always downloaded on demand.

### Why This Architecture

- **Correctness for ROM hacks:** Damage calc uses each hack's actual data, not retail data with wrong values
- **Handles new content cleanly:** Hack-native Pokémon/moves/routes are just data, no special override semantics
- **Negligible data sizes:** 200-400 KB gzipped per game; total app data footprint stays small
- **Updates without app store releases:** New ROM hacks ship by merging a PR
- **Community contributions:** PR workflow with CI schema validation
- **Free hosting:** GitHub raw content (Cloudflare in front when scale demands)
- **License consistency:** GPL 3.0 data repo matches GPL 3.0 app

---

## 6. Phased Roadmap

### MVP — Ship It
**Goal:** Playable GBA emulator + functional Nuzlocke tracker. Replaces emulator + spreadsheet.

- mGBA GBA emulation
- ROM load, play, save (in-game + 5 manual save states + RRSS)
- Fast forward (2/4/6/8/10x)
- On-screen controls + basic controller support
- Save file parsing: full auto-import (party + box, all fields — see §13)
- Nuzlocke tracker: run setup, encounter logging, team/box/graveyard, death logging
- IV checker, EV display, stat viewer, nature, met location auto-match
- Badge tracker + level cap tracker
- Line Builder: current MVP includes target selection, saved lines, and manual step composition
- Run history + basic metrics
- Bundled game data and sprite display for MVP games
- Game data: Emerald + FireRed only
- Dark mode UI, portrait + landscape layouts

**Platform targets:** Android first → Windows → iOS

---

### Phase 1 — Feature Complete
**Goal:** Full feature set. All launch game data. DS on all platforms.

- melonDS DS emulation (all three platforms)
- Full controller support with mapping UI
- All Phase 1 game data (§12.2)
- Emerald Kaizo + Platinum Kaizo data
- Damage calculator with boss presets
- Line Builder refinements (§9), including generated-threat workflow if it remains in scope
- Run export (share card + JSON)
- Route notes, Move/PP tracker, Shiny counter
- Wedlocke run mode
- DS dual-screen layout options
- Frame advance
- iCloud + Google Drive backup
- Home screen widgets (§10)
- Supporter Pack IAP

---

### Phase 2 — Community & Content
**Goal:** Grow the audience. Scale data via community.

- Soul Link run mode (local/manual + same-WiFi sync)
- Custom ROM hack data import + public GitHub registry
- Randomizer mode + Universal Randomizer log import
- OBS overlay (local HTTP server)
- Challenge mode presets
- Post-launch ROM hack wave (Radical Red, FireRed Rocket Edition)

---

### Phase 3 — Platform Expansion
**Goal:** Full classic era. Expanded platforms.

- SameBoy GB/GBC — Gen 1 + Gen 2 full support
- Gen 5 DS (Black/White/B2W2)
- macOS port
- Apple Watch companion (iOS)
- Gen 1/2 save parsing (§13.3)

---

## 7. Feature Specification — Emulator Core

### 7.1 Playback Controls
- Fast Forward: 2x / 4x / 6x / 8x / 10x — toggle or hold, assignable to controller
- Pause / Resume
- Frame advance (single step)
- DS screen layout: stacked, side-by-side, swappable, single screen focus

### 7.2 Save System

**Auto-save:** Off / 5 / 10 / 15 min interval. Emulator state snapshot, separate from manual slots.

**Manual save states:** 5 named slots with frame thumbnail and timestamp.

**Run Reset Save State (RRSS):**
- One-time capture via "Lock In Run" at run start
- Immutable after creation
- "Reset to Start" instantly returns to this state
- Confirmation: *"This save state cannot be changed. Lock it in?"*
- Primary use: Kaizo quick resets

### 7.3 Controller Support
- Full button mapping UI
- Default presets: Xbox, PlayStation, Switch Pro, 8BitDo
- Keyboard mapping (Windows)
- On-screen touch controls — configurable opacity, size, layout
- Turbo button support
- Controller hot-plug

---

## 8. Feature Specification — Nuzlocke Tracker

### 8.1 Run Setup
Game, run mode (Classic / Wedlocke / Soul Link / Custom), rules (Dupes Clause, Species Clause, Shiny Clause, Gift allowed, Legendary allowed, Level cap enforcement), randomizer mode toggle, seed log, run name.

### 8.2 Encounter Tracker
Routes pre-populated from game data. Statuses: `Unclaimed` / `Encountered` / `Missed` / `Blackout Skip` / `Dupes Clause`

**Logging:** Select species (filtered to route-legal), nickname, level, ball. Dupes clause check. Confirm → team or box.

**Auto-import from save:** "Sync from Save" reads save buffer, auto-matches caught Pokémon to routes via met location ID. Eliminates manual encounter logging.

### 8.3 Team & Box Management
**Team (6 max):** Sprite, nickname, level, held item, HP, status. Drag to reorder. Expand for moveset + PP + notes.  
**PC Box:** All living benched Pokémon. Filter: alive / fainted / released.  
**Graveyard:** All fainted, permanent. Shows nickname, species, level at death, cause, location, timestamp. Undoable within session.

### 8.4 Death Logging
"Mark as Fainted" → cause of death (free text or quick-select) → optional location → moves to graveyard.

### 8.5 Damage Calculator
**Inputs:** Attacker + Defender (species, level, stat stage, item, ability, current HP) + Move  
**Output:** Damage range, % HP, OHKO/2HKO/3HKO label, hit probability  
**Quick calc:** Pre-populate from active team in one tap  
**Boss presets:** Gym leaders, rivals, E4 with known sets  
**Gen accuracy:** Gen 3 vs Gen 4 physical/special split handled

### 8.6 Badge Tracker
Visual badge case per game. Tap to mark earned. Unlocks level cap tier.

### 8.7 Level Cap Tracker
Per-badge cap displayed when enforcement is on. Warning indicator on Pokémon at or above cap. User-overridable per badge (for ROM hacks).

### 8.8 Pokémon Stat Viewer
Populated automatically from save parse. Per Pokémon:

- **IVs:** All 6 (0–31), color-coded (0 = red → 31 = gold)
- **EVs:** All 6 (0–255), progress bar toward 252 cap, total vs 510 max, warning if over-trained
- **Nature:** Name + stat modifier (+Atk / -SpA style display)
- **Ability:** Active slot
- **Actual stats:** Real in-game computed values (not estimated)
- **Base stats:** From PokeAPI, shown alongside for comparison
- **Hidden Power:** Type and power calculated from IVs
- **Met location:** Auto-matched route name
- **Shiny indicator**
- **Friendship value**
- **Pokérus status**

All data is read-only — sourced from save file. No manual entry.

### 8.9 Run Metrics
Encounters (attempted / claimed / missed / duped), deaths by cause, furthest progress, active play time, unique species used, longest surviving member, routes completed vs total.

### 8.10 Run History
All runs local. Status: In Progress / Completed / Failed. Read-only summary for past runs. Resume for active.

### 8.11 Run Export
**Share Card:** Image recap — team sprites, graveyard highlights, stats. Social-ready.  
**JSON Export:** Full structured run data.

### 8.12 Move / PP Tracker
Per Pokémon: PP per move, quick decrement, "Healed at Center" resets all. Syncs from save.

### 8.13 Route Notes
Free-form per route. Game-level scope (persists across runs). Tags: warning / tip / reminder.

### 8.14 Shiny Counter
Global per run + optional per-route tracking.

### 8.15 Run Modes
**Classic:** Standard rules  
**Wedlocke:** M/F pairs — partner is benched on death  
**Soul Link:** Two-player linked catches by route. Phase 1: manual/JSON or same-WiFi sync  
**Custom:** User-defined toggles

---

## 9. Feature Specification — Line Builder

### 9.1 Purpose
*"Who do I bring to this fight, in what order, and why?"* Uses actual in-game stats from save parse — not estimates.

### 9.2 Flow
1. **Sync from save** — party + box with actual stats, IVs, nature, moves
2. **Select squad** — 1–6 from full roster
3. **Set enemy** — boss preset or manual
4. **Calc matrix** — damage dealt %, damage taken %, OHKO/2HKO, speed comparison for every matchup
5. **Generate line** — ranked: lead → mid → closer
6. **Save line** — named reference per run ("vs Roxanne", "vs Champion Steven")

### 9.3 Ranking Logic
```
threat_score(mine, theirs) = damage_dealt_pct / max(damage_taken_pct, 1)
```
Lead = best score that also survives first hit. Mid = best coverage for lead's weak matchups. Closer = highest damage against weakened threats. Pure deterministic math, all local.

---

## 10. Feature Specification — Widgets

Data-only home screen widgets. Updated from SQLite snapshot on save events.

**Types:** Run Status (medium) / Graveyard (small-medium) / Next Boss (small) / Run Streak (small)

**iOS:** WidgetKit + App Group. **Android:** App Widget + shared storage. **Windows:** lower priority.

3 widget themes in Supporter Pack. Default always free.

---

## 11. Post-Launch Features

- **Phase 3:** GB/GBC (SameBoy), Gen 1/2/5, macOS, Apple Watch
- **Phase 2:** Soul Link sync, ROM hack registry, Randomizer log import, OBS overlay, challenge presets
- **Post Phase 3:** Community run sharing (requires moderation infra — defer until community justifies it)

---

## 12. Game Data

All original data compiled from Bulbapedia, Smogon, decomp projects.

### 12.1 MVP Games
Pokémon Emerald (GBA Gen III), Pokémon FireRed (GBA Gen III) — full encounter tables, boss sets, level caps.

### 12.2 Phase 1 Launch Games

| Game | Gen | Platform |
|---|---|---|
| Pokémon Emerald, FireRed, LeafGreen, Ruby, Sapphire | III | GBA |
| Pokémon Diamond, Pearl, Platinum, HeartGold, SoulSilver | IV | DS |

All with full encounter tables, boss sets, level caps.

### 12.3 ROM Hacks

| ROM | Base | Phase |
|---|---|---|
| Emerald Kaizo | Emerald | Phase 1 |
| Platinum Kaizo | Platinum | Phase 1 |
| Radical Red | FireRed | Phase 2 |
| FireRed Rocket Edition | FireRed | Phase 2 |
| Crystal Clear | Crystal | Phase 3 |

### 12.4 Phase 3 Games
GB/GBC: Red, Blue, Yellow, Gold, Silver, Crystal  
DS Gen 5: Black, White, Black 2, White 2

### 12.5 Data File Format

```
assets/game_data/
  emerald/
    metadata.json     # name, gen, region, badge names
    routes.json       # routes, encounter slots, methods, met location IDs
    bosses.json       # gym leaders, rivals, E4, Champion full sets
    level_caps.json   # per-badge cap values
  emerald_kaizo/
    metadata.json     # overrides only
    routes.json       # changed routes only — inherits rest from emerald
    bosses.json       # full file (all bosses changed)
    level_caps.json   # adjusted caps
```

---

## 13. Save File Parsing — Technical Reference

Save data accessed via mGBA/melonDS FFI — no process attachment, no cross-process scanning. All parsing is in-process Dart.

```c
void* mCoreSaveData(struct mCore* core);
size_t mCoreSaveDataSize(struct mCore* core);
```

### Auto-Import: Full Field List

Every field below is populated automatically. **User never manually enters stats.**

| Field | Source |
|---|---|
| Species | Substructure G (Gen3) / Block A (Gen4) |
| Nickname | Bytes 0x08–0x11, 6-bit encoded |
| Level | Battle stats block |
| Current HP | Battle stats block |
| Max HP, Atk, Def, Spd, SpA, SpD | Battle stats block — actual in-game values |
| IVs (all 6) | Substructure M / Block D — 5 bits per stat |
| EVs (all 6) | Substructure E / Block C — 1 byte per stat, 0–255 |
| Nature | `PV % 25` → 25-nature table |
| Ability slot | Substructure M / Block A |
| Moves (4) + current PP | Substructure A / Block B |
| Held item | Substructure G / Block A |
| Met location | Substructure M origins bitfield → route name via game data JSON |
| Shiny flag | `(OT_ID XOR SID XOR upper16(PV) XOR lower16(PV)) < 8` |
| Friendship | Substructure G / Block A |
| Pokérus | Substructure M |
| Ball caught in | Origins bitfield |
| Level met | Origins bitfield |

---

### 13.1 Gen 3 (GBA)

**Save size:** 128KB. Two alternating 57,344-byte blocks.  
**Party offset:** Block base + `0x0238` | **Party count:** Block base + `0x0234`  
**Struct size:** 100 bytes

```
Offset  Size  Field
0x00    4     Personality Value (PV)
0x04    4     OT ID (upper 2 bytes = secret ID, lower 2 = public ID)
0x08    10    Nickname (6-bit encoded)
0x14    7     OT Name
0x1C    2     Checksum
0x20    48    Encrypted substructure block (4 × 12 bytes, order = PV % 24)
0x50    2     Status condition
0x52    1     Level
0x54    2     Current HP
0x56    2     Max HP
0x58    2     Attack
0x5A    2     Defense
0x5C    2     Speed
0x5E    2     Sp. Attack
0x60    2     Sp. Defense
```

**Decryption key:** `PV XOR OT_ID` (32-bit XOR, 4-byte chunks across 48 bytes)  
**Shuffle order:** `PV % 24` → lookup table → [G, A, E, M] substructure positions

**Substructure G — Growth (12 bytes):**
```
0x00  2  Species ID
0x02  2  Held Item ID
0x04  4  Experience
0x09  1  Friendship
```

**Substructure A — Attacks (12 bytes):**
```
0x00–0x07  Move IDs 1–4 (2 bytes each)
0x08–0x0B  Current PP for moves 1–4
```

**Substructure E — EVs (12 bytes):**
```
0x00  HP EV   0x01  Atk EV   0x02  Def EV
0x03  Spd EV  0x04  SpA EV   0x05  SpD EV
0x06–0x0B  Contest stats (Coolness, Beauty, Cuteness, Smartness, Toughness, Sheen)
```

**Substructure M — Misc (12 bytes):**
```
0x00  1  Pokérus
0x01  1  Met Location ID → maps to route name in routes.json
0x02  2  Origins bitfield (level met [0:6], game [7:10], ball [11:14], OT gender [15])
0x04  4  IVs bitfield:
           HP [0:4], Atk [5:9], Def [10:14], Spd [15:19], SpA [20:24], SpD [25:29]
           IsEgg [30], Nicknamed [31]
0x08  4  Ribbons
```

**Derived values:**
```dart
// Nature
const natures = ['Hardy','Lonely','Brave','Adamant','Naughty','Bold','Docile',
  'Relaxed','Impish','Lax','Timid','Hasty','Serious','Jolly','Naive','Modest',
  'Mild','Quiet','Bashful','Rash','Calm','Gentle','Sassy','Careful','Quirky'];
final nature = natures[pv % 25];

// Shiny
bool isShiny(int pv, int otId) =>
  ((otId >> 16) ^ (otId & 0xFFFF) ^ (pv >> 16) ^ (pv & 0xFFFF)) < 8;

// Hidden Power type
int hpType(IVs iv) =>
  ((iv.hp&1) + (iv.atk&1)*2 + (iv.def&1)*4 + (iv.spd&1)*8 +
   (iv.spa&1)*16 + (iv.spd2&1)*32) * 15 ~/ 63;

// EV validation
bool evLegal(List<int> evs) =>
  evs.reduce((a,b) => a+b) <= 510 && evs.every((e) => e <= 255);

// Checksum
bool checksumValid(Uint8List decrypted, int stored) {
  int sum = 0;
  for (int i = 0; i < 48; i += 2) sum += decrypted[i] | (decrypted[i+1] << 8);
  return (sum & 0xFFFF) == stored;
}
```

**PC Box:** Sectors 5–13. Box struct: 80 bytes (no battle stats). 14 boxes × 30 = 420 Pokémon max.

---

### 13.2 Gen 4 (DS)

**Save size:** 512KB. Block size: `0xCF2C` (DPPt) / `0x12310` (HGSS)  
**Party offset:** `0x0088` (DPPt) / `0x0098` (HGSS)  
**Struct size:** 236 bytes

```
0x00    4     Personality Value
0x04    2     Checksum
0x08    128   Encrypted block (8 substructures × 16 bytes, PV % 24 shuffle)
0x88    100   Battle stats block (unencrypted in party)
```

**Battle stats (0x88):** Same layout as Gen 3 battle stats — Level at 0x02, Current HP at 0x04, then Max HP, Atk, Def, Spd, SpA, SpD as 2-byte values.

**Encryption key:** `PV` only (not XOR'd with OT_ID — Gen 4 change from Gen 3)

**Block A (16 bytes):** Species ID, Held Item ID, Experience, Friendship, Ability slot, HP EV, Atk EV, Def EV, Spd EV  
**Block B (16 bytes):** SpA EV, SpD EV, Move IDs 1–4, PP 1–4, PP Ups 1–4  
**Block D (16 bytes):** IVs bitfield (same 30-bit layout as Gen 3), Hoenn ribbons, Met location, Origins

**Met location:** 1 byte in Block D → maps to DPPt or HGSS location ID table in routes.json

**PC Box:** Offset `0xA000` (DPPt — verify HGSS against decomp). 136-byte box structs. 18 boxes × 30 = 540 max.

Full block layout reference: https://bulbapedia.bulbagarden.net/wiki/Save_data_structure_(Generation_IV)

---

### 13.3 Gen 1 / Gen 2 (Phase 3)

**Gen 1:** No encryption. Party at `0x2F2C`. 44-byte structs. IVs packed as 2 bytes (4 bits per stat). EVs as 16-bit values (0–65535).  
**Gen 2:** Adds held item + friendship byte. Offsets differ between G/S and Crystal.

Full structs deferred to Phase 3 kickoff.  
Reference: https://bulbapedia.bulbagarden.net/wiki/Pok%C3%A9mon_data_structure_(Generation_I)

---

### 13.4 Dart Parsing Layer

```dart
class Gen3SaveParser {
  final Uint8List saveBytes;
  Gen3SaveParser(this.saveBytes);

  int get _activeBlockBase {
    final c0 = _u32(0xFFC);
    final c1 = _u32(0xE000 + 0xFFC);
    return c0 > c1 ? 0 : 0xE000;
  }

  List<ParsedPokemon> get party {
    final base = _activeBlockBase;
    final count = saveBytes[base + 0x0234];
    return List.generate(count, (i) {
      final off = base + 0x0238 + (i * 100);
      return _parsePokemon(saveBytes.sublist(off, off + 100));
    });
  }

  ParsedPokemon _parsePokemon(Uint8List d) {
    final pv    = _u32at(d, 0x00);
    final otId  = _u32at(d, 0x04);
    final key   = pv ^ otId;

    // Decrypt
    final enc = d.sublist(0x20, 0x50);
    final dec = Uint8List(48);
    for (int i = 0; i < 48; i += 4) {
      final w = _u32at(enc, i) ^ key;
      dec[i]=w&0xFF; dec[i+1]=(w>>8)&0xFF;
      dec[i+2]=(w>>16)&0xFF; dec[i+3]=(w>>24)&0xFF;
    }

    // Checksum
    int sum = 0;
    for (int i = 0; i < 48; i += 2) sum += dec[i] | (dec[i+1] << 8);
    if ((sum & 0xFFFF) != _u16at(d, 0x1C)) throw BadEggException();

    // Substructure positions
    final idx = _substructureOrder(pv % 24); // [gIdx, aIdx, eIdx, mIdx]
    final g = dec.sublist(idx[0]*12, idx[0]*12+12);
    final a = dec.sublist(idx[1]*12, idx[1]*12+12);
    final e = dec.sublist(idx[2]*12, idx[2]*12+12);
    final m = dec.sublist(idx[3]*12, idx[3]*12+12);

    // IVs
    final ivBits = _u32at(m, 0x04);
    final ivs = StatBlock(
      hp:  (ivBits>>0)&0x1F,  atk: (ivBits>>5)&0x1F,
      def: (ivBits>>10)&0x1F, spd: (ivBits>>15)&0x1F,
      spa: (ivBits>>20)&0x1F, spdef:(ivBits>>25)&0x1F,
    );

    // EVs
    final evs = StatBlock(
      hp: e[0], atk: e[1], def: e[2],
      spd: e[3], spa: e[4], spdef: e[5],
    );

    return ParsedPokemon(
      speciesId:    _u16at(g, 0x00),
      heldItemId:   _u16at(g, 0x02),
      friendship:   g[0x09],
      moves:        [_u16at(a,0),_u16at(a,2),_u16at(a,4),_u16at(a,6)],
      currentPP:    [a[8], a[9], a[10], a[11]],
      evs: evs, ivs: ivs,
      nature:       pv % 25,
      metLocationId: m[0x01],
      isShiny:      ((otId>>16)^(otId&0xFFFF)^(pv>>16)^(pv&0xFFFF)) < 8,
      isEgg:        (ivBits >> 30) & 1 == 1,
      level:        d[0x52],
      currentHp:    _u16at(d, 0x54),
      stats: StatBlock(
        hp:    _u16at(d, 0x56), atk:  _u16at(d, 0x58),
        def:   _u16at(d, 0x5A), spd:  _u16at(d, 0x5C),
        spa:   _u16at(d, 0x5E), spdef:_u16at(d, 0x60),
      ),
      nickname: _decodeGen3String(d.sublist(0x08, 0x12)),
    );
  }

  int _u32(int o) => _u32at(saveBytes, o);
  static int _u32at(Uint8List d, int o) =>
    d[o]|(d[o+1]<<8)|(d[o+2]<<16)|(d[o+3]<<24);
  static int _u16at(Uint8List d, int o) => d[o]|(d[o+1]<<8);
}
```

---

## 14. UI / UX Principles

- **Emulator-first:** game screen is primary. Tracker never forces itself over gameplay.
- **One-handed use:** tracker panel fully operable with thumb in portrait.
- **Dark mode default** — the audience plays GBA games at night.
- **Local-first:** no ads, no accounts, no telemetry. Backup is opt-in.
- **Accessible:** AA contrast minimum, scalable text, screen reader support on tracker UI.

### Layout Modes

| Context | Layout |
|---|---|
| Phone portrait | Game top ~60%, tracker panel as bottom sheet with collapsed handle |
| Phone landscape | Game full width, tracker as swipe-in drawer from right edge |
| Tablet / Windows | Side-by-side split — game left, tracker right, resizable |
| DS portrait | Top screen → gap → bottom screen, tracker behind handle |
| DS landscape | Both screens side-by-side, tracker drawer from edge |

### Tracker Panel
- Persistent collapsed handle showing death count + current route status
- Expands without covering game screen (portrait)
- Tab bar inside: **Team** / **Routes** / **Graveyard** / **Stats** / **Lines**
- Optional notification indicator on handle when a route is claimable

---

## 15. Out of Scope

| Feature | Reason | Target |
|---|---|---|
| Auto-detection via live memory scanning | Platform-restricted, fragile | Post-launch evaluation |
| Gen 1/2 GB/GBC | Separate core integration | Phase 3 |
| Gen 5 DS | DS expansion wave | Phase 3 |
| macOS | Low priority, easy from iOS | Phase 3 |
| Soul Link server sync | Requires server infra | Phase 2 local only |
| Video recording / streaming SDK | OBS overlay covers creators | Post Phase 3 |
| Web platform | Not needed | Never unless demanded |
| Community run sharing backend | Requires moderation infra | Post Phase 3 |
| Apple Watch | iOS-only, niche | Phase 3 |

---

## 16. Open Questions

### Resolved Scope Clarifications

- **Line Builder MVP scope:** Current MVP implementation includes the Lines tab and manual line builder flow. Phase 1 now covers refinements such as generated threat scoring and broader boss preset polish rather than first introduction.

### Still Open

1. **HGSS PC box offset:** Verify `0xA000` against HGSS decomp before Gen 4 box parsing. Conflicting community references.

2. **Share card rendering:** Evaluate `dart:ui` Picture.toImage() vs `flutter_screenshot` at social resolution.

3. **Soul Link same-WiFi scope:** Evaluate Bonjour/mDNS local discovery for Phase 1 vs deferring to Phase 2.

4. **ROM hack data pipeline:** Emerald Kaizo + Platinum Kaizo are dedicated data tasks, separate from engineering. Est. 3–5 days each.

5. **PokeAPI at scale:** Monitor rate limit impact. Static JSON snapshot is the fallback if batched fetch is too slow.

6. **Supporter Pack asset GPL treatment:** Confirm with a software attorney that cosmetic art assets can be treated as separate artistic works not subject to GPL copyleft before App Store submission.

7. **Gen 3 substructure shuffle table:** The 24-permutation GAEM ordering table must be hardcoded in the parser.  
   Reference: https://bulbapedia.bulbagarden.net/wiki/Pok%C3%A9mon_data_structure_(Generation_III)

---

*End of specification.*
