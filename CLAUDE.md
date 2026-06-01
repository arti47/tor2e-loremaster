# TOR2E Loremaster Companion App

A single-file HTML5 companion app for the **The One Ring 2nd Edition** RPG, designed for Loremasters to run encounters, manage party state, track the Eye of Mordor, resolve delving journeys, roll campaign loot, and log sessions.

---

## Local Development & Run

```bash
# Serves the app locally (optional, for PWA/offline testing)
python3 -m http.server 8000

# Open in browser
open http://localhost:8000/tor2e-loremaster/loremaster-tracker.html
```

---

## File Layout

```
tor2e-loremaster/
├── loremaster-tracker.html   # Canonical edit target - all edits happen here first
├── index.html               # Mirror of loremaster-tracker.html for hosted deploys
├── sw.js                     # Cache-first service worker for PWA caching
├── manifest.json            # PWA configuration manifest
├── icon.svg                 # Scalable gold-and-red book/eye SVG icon
└── CLAUDE.md                # This file (development guidelines and commands)
```

**Sync copy rule**: after editing `loremaster-tracker.html`, always sync to `index.html`:
```bash
cp loremaster-tracker.html index.html
```

---

## Technical Standards

1. **Stack**: Pure HTML5, CSS, and Vanilla JavaScript. No frameworks or external dependencies.
2. **Offline-first**: Stored character/campaign data uses `localStorage` keys:
   - `tor2e-lm-campaign-v1` (active campaign, calendar, and notes)
   - `tor2e-lm-encounter-v1` (active combat tracker list, rounds, and player stances)
   - `tor2e-lm-eye-v1` (eye awareness level, history, region, and chosen revelation table)
   - `tor2e-lm-journey-v1` (active delving journey planner, logs, progress, and options)
   - `tor2e-lm-party-v1` (uploaded player character JSON arrays and Dwarven Moria-Madness flaws)
   - `tor2e-lm-loot-v1` (treasure hoard and lore rumours history)
   - `tor2e-lm-bestiary-v1` (custom reusable adversary templates — "My Bestiary")
   - `tor2e-lm-presets-v1` (saved encounter lineups)
   - `tor2e-lm-theme` (light/dark user preference)
3. **PWA Integration**: Implemented using a service worker that caches pages for complete offline capability on mobile viewports.
4. **Integration Pattern**: Import player state by parsing player-exported JSONs; export custom gear by generating compliant JSON copy blocks.
5. **Moria rules toggle**: `state.campaign.moriaMode` gates the *Moria — Through the Doors of Durin* supplement rules. **Convention: tag every Moria-only UI element with `class="moria-only"`** — a single `refreshMoriaUI()` (called from `renderAll`) shows/hides them all and keeps the header button (`#moria-btn`) and the Campaign-tab checkbox in sync. Do NOT add bespoke per-element show/hide for Moria visibility; just add the class. Moria-only today: Orc Band generator, Water Peril button, Moria Chamber Generator card, Loremaster Rumours card, the Eye "Moria Noise & Discovery" hooks block, and the Drums-in-the-Deep hook. Either/or relabels (journey card title, Moria-vs-standard journey buttons, revelation-table selector) stay in `renderEyeTab`. Enabling Moria also defaults the Eye to Dark Land (Hunt Threshold 14) if still on the core default.

---

## Features Reference

### ⚔️ Encounter & Combat Stances
- **Stance & Initiative Tracker**: Computes target TNs (Strength TN + Parry) and sorts the initiative order dynamically (Forward/Open -> Adversaries -> Defensive/Rearward).
- **Stance Attack Modifiers**: Enemy attacks auto-evaluate mod types (Favoured vs Forward targets, Ill-Favoured vs Defensive targets).
- **Special Damage Toolbar**: GMs spend 1 enemy Hate/Resolve to execute **Heavy Blow** ($+AL$ damage), **Break Shield**, **Seize** (grappled), or **Fiery Blow** (+2 Fatigue, -1d Protection).

### 👥 Party Status & Corruption
- **Shadow Check Roller**: Group shadow tests with custom failure shadow multipliers for **Anguish/Desecration** (Dread vs Heart TN, Valour dice), **Misdeeds** (Greed vs Wits TN, Wisdom dice), and **Sorcery** (Sorcery vs Wits TN, Wisdom dice). Failures present a `+Shadow` button to add points to character files.
- **Moria-Madness**: Dwarven corruption tracker flaws checklist (Distracted -> Mistrustful -> Blinded -> Jealous).

### 👁️ Eye & Journey Resolver
- **Moria Journey Resolver**: planner supporting base distance, path complexity increases (+50% level shifts or +100% windy paths), forced march (double progress), and deeps travel test penalties (-1d Travel). Auto-rolls Guide's Travel skill, advances progress, and resolves d12 **Water Perils** and **Moria Journey Events**.
- **Landmark Seeking**: Scan tests vs renown (Famous, Obscure -1d, Hidden -2d) to discover legendary locations.
- **Revelation Table Dropdown**: Selector to switch active tables (Core Rules, Dire Portents, Terrors of the Dark, Orc Assault, or Ghâsh! Balrog table).

---

## Roadmap (candidate features)

Brainstormed 2026-06-01. Not yet built — listed for prioritisation. Checkboxes flip to `[x]` (with a Changelog entry) as they ship.

### ⭐ Top picks (best impact-to-effort)
- [x] **Persistent custom-adversary library** — ✅ done 2026-06-01. Custom foes save to `tor2e-lm-bestiary-v1` and appear in Add Adversaries under "My Bestiary" with a CUSTOM tag + 🗑 delete.
- [x] **Encounter presets** — ✅ done 2026-06-01. Save the current lineup by name to `tor2e-lm-presets-v1`; Load restores fresh instances (full End/Hate, conditions cleared); Delete removes.
- [x] **Party-wide XP / advancement awards** — ✅ done 2026-06-01. Party tab card awards SP/AP/FP/Treasure to all heroes; 📜 End Session (+3/+3) quick button; pairs with the per-hero ⬇ JSON export.
- [x] **Campaign backup/restore** — ✅ done 2026-06-01. Campaign tab card exports ALL LM state to one JSON (with `_app` marker) and restores it with confirmation.

### ⚔️ Combat & adversaries
- [ ] **Stat-block paste/import** — parse the Markdown emitted by the `tor2e-adversary-statblock` skill (NotebookLM) into a combatant; closes the loop between the two tools.
- [ ] **Unified initiative** — fold adversaries into the turn order (currently the initiative tracker sorts only imported players by stance; foes aren't listed).
- [ ] **Adversary scaling** — adjust AL/Endurance/count by party size.
- [ ] **Fell-ability / Hate budget reminders** — visual cue when a foe can/can't afford a special; per-round Hate reset.
- [ ] **Opening Volleys phase** — quick ranged-exchange step before melee.
- [ ] **Moria Battle / Clash tracker (LM side)** — run the war-party clash for a group game (the player app has the solo Band/Battle system).

### 📅 Campaign management
- [ ] **In-fiction calendar** — track the date (Shire/Steward's Reckoning), auto-flag Yule (→ Fellowship Phase), derive **season** → feed journey event modifiers.
- [ ] **Patron tracker** — per-patron card (agenda, sanctuary, standing, undertakings used), distinct from the freeform NPC ledger.
- [ ] **Fellowship Phase manager (party-wide)** — table-side mirror of the player wizard: each hero's undertaking, Heal Scar, Open a New Sanctuary, etc.

### 🗺️ Journey & Eye depth
- [ ] **Core-Rules overland journey mode** — hex-based Wilderland journey with role fatigue and the standard Journey Events table (current resolver is Moria-miles only); switchable by region.
- [ ] **Region-specific event tables** — Mirkwood / Rivendell / Lost Realm supplements, via a dropdown like the Revelation selector.
- [ ] **Eye / Hunt campaign log** — timeline of Eye rises/falls and which Revelations fired across sessions.

### 🎲 Reference & generators
- [ ] **Quick reference cards** — tap-to-open cheat cards for Conditions, Stances, Combat Tasks, the TN ladder.
- [ ] **Random generators panel** — NPC name (by culture), weather, settlement, plot hook/rumour seed (generalises the existing rumour/chamber/orc-band generators).
- [ ] **General LM dice roller** — plain Feat + Success die vs TN for ad-hoc NPC/adversary tests (current rolls are tied to specific foes/group checks).

### ✨ Nice-to-have
- [ ] **Map / handout upload** — drop an image (hex map, location art) onto a tab for reference.
- [ ] **Big-screen "table mode"** — high-contrast, large-text encounter view for casting to a TV in person.
- [ ] **Print stylesheet** — print the encounter or party dashboard.

---

## Workflow for Claude

1. **Edit `loremaster-tracker.html` first** (canonical), then mirror: `cp loremaster-tracker.html index.html`.
2. **Always update this CLAUDE.md after changing code** — add a Changelog entry and refresh any affected section.
3. Bump `CACHE_NAME` in `sw.js` if deploying so installed PWAs pull the new shell (currently `tor2e-lm-v2`).

---

## Changelog

- **2026-06-01 — Fixed adversary attack Target Number (RAW).** The Adversary Attack Roll computed the hero's TN as `(hero.strTN||14) + (hero.parry||0)`, which is wrong: in TOR2e the TN of an adversary's attack against a hero is the **hero's Parry score** (a full TN — culture base `Wits + 10/12/14` + shield), *not* Strength TN (Strength TN governs the hero's own attacks vs an enemy's Parry). Also, the targeted hero's stance was auto-applied as a Favoured/Ill-favoured **Feat die**; RAW instead adds/removes a **Success die** (Forward +1d, Defensive −1d). Fixes: `heroTN = hero.parry || 14` in all three sites (attack-roll target selector, `renderStanceTracker`, `renderStanceGrid`); the attack modal now auto-fills the **Success Dice Mod** from stance (`STANCE_DICE = {Forward:1, Defensive:-1}`) via a `data-dice` option attr + `onAttackTargetChange`, and the Roll Modifier dropdown defaults to Normal (kept for genuine Favoured/Ill effects like special abilities/conditions). Display labels now read "Parry"/"Attack TN". The lone surviving `hero.strTN` use (`hero` Strength-skill tests) is correct and untouched. JS parses clean; synced to `index.html`. Cache stays `tor2e-lm-v2`.
- **2026-06-01 — Interactive NPC Ledger.** The ledger was previously a read-only list (rows did nothing). Now every row is tappable → `openNPCDetail(ref)` modal. **Custom NPCs:** edit name/role/features/notes + new **Location**, **Attitude** (Friendly/Neutral/Hostile), **Status** (Alive/Dead) fields; Delete; and **⚔️ To Encounter** → `addNPCToEncounter()` picks any stat block (built-in or My Bestiary, 44+) to represent the NPC and pushes a combatant under the NPC's name (`npcStatBlock` records the source). **Lore NPCs** (now hoisted to `const LORE_NPCS`) stay **read-only reference** — detail view + To-Encounter only, no edit/delete. List rows show attitude + 📍location tags and strike-through + 💀 for dead. Add-NPC form gained Location + Attitude inputs (`addNewNPC` writes `location`/`attitude`/`status:'alive'`). Verified in preview (edit→dead, lore read-only, send-to-combat as Great Orc Chief); no console errors; synced to `index.html`.
- **2026-06-01 — Centralized Moria-rules toggle.** Replaced the scattered per-element Moria show/hide with one convention: `class="moria-only"` + `refreshMoriaUI()` (runs in `renderAll`). Tagged the previously-ungated Moria features — **Orc Band** generator and the Eye **"Moria Noise & Discovery"** hooks block (+ Drums) — plus moved the chamber card, rumours card, and water-peril button onto the class (removing their bespoke JS in `renderEyeTab`/`renderLootTab`). Added a prominent **header toggle** (`#moria-btn`, "⛏️ Moria: On/Off", gold when on) that flips `toggleMoriaMode` and stays synced with the Campaign-tab checkbox. Enabling Moria now also sets the Eye to **Dark Land (Hunt Threshold 14)** when still on the core default, and `renderEyeTab` syncs the region `<select>`. Verified in preview (6 moria-only elements toggle correctly both ways, threshold 18↔14, revelation resets to core on disable); no console errors; synced to `index.html`.
- **2026-06-01 — Top-pick feature batch (4 features).** All four roadmap top picks shipped:
  - **Custom-adversary library** — `state.bestiary` (`tor2e-lm-bestiary-v1`). The Custom Foe modal gained a "💾 Also save to My Bestiary" checkbox; saved templates (ADVERSARY_DB-shaped, `_lib:true`, name collision → " (Custom)") appear in the Add Adversaries list under the new **My Bestiary** category with a CUSTOM tag + 🗑 delete (`deleteBestiaryEntry`). `allAdversaries()` = built-ins + bestiary; `addAdversaryToCombat`/`populateAdvDb` use it. onclick names are single-quote-escaped.
  - **Encounter presets** — `state.presets` (`tor2e-lm-presets-v1`). New Encounter card: `saveEncounterPreset` (modal name input), `loadEncounterPreset` (deep-copies with fresh ids, `enduranceCur=Max`, `hateCur=Max`, conditions/target cleared, round→1; confirms if combatants exist), `deleteEncounterPreset`. `renderPresetSelect()` runs inside `renderEncounterTab`.
  - **Party-wide XP awards** — Party tab card: `awardPartyXP()` adds SP/AP/FP/Treasure to every loaded hero (writes `skillPts`/`advPts`/`fellowship`/`treasure` to match the player schema), shows a per-hero result log, resets inputs. `endSessionParty()` → confirm → `awardPartyXP({sp:3,ap:3})` per RAW p.55.
  - **Campaign backup/restore** — Campaign tab card: `exportCampaignBackup()` writes one JSON (`_app:'tor2e-loremaster'`, `_version:1`) with campaign/party/encounter/eye/journey/loot/bestiary/presets/theme; `importCampaignBackup()` validates the marker, confirms, replaces all state, re-inits theme + adv DB + treasure index + renderAll, and resets the file input.
  - Verified each in preview (bestiary add + filter, preset save/load reset, XP math + End-Session stacking, backup round-trip); no console errors; synced to `index.html`. Cache stays `tor2e-lm-v2`.
- **2026-06-01 — Per-hero re-export + robust tab switching.** (1) Added an **⬇ JSON** button to each Party dashboard row (`exportHero(idx)`) that downloads the Loremaster's copy of that hero as JSON, so the GM can hand a modified sheet (e.g. after a `+Shadow` apply) back to the player to re-import (☰ Menu → Import on the player tracker). (2) `showTab()` previously highlighted the active tab by matching a 3-letter `innerText` prefix — fragile if tab labels ever collide. Tabs now carry explicit `data-tab` attributes and `showTab` matches `.tab[data-tab="..."]`. Verified in preview (all 5 tabs activate the correct button+panel; export produces correct filename/content); synced to `index.html`. Cache stays `tor2e-lm-v2` (not yet deployed).
- **2026-06-01 — Shadow tracking fix + Party dashboard Shadow column.** `applyShadowDirect()` (the `+Shadow` button after a failed group Shadow test) was writing `hero.shadowPoints`, but imported player JSON uses `hero.shadow` (per the player tracker's `DEFAULT_CHARACTER`) — so the mutation hit a dead field. Now writes `hero.shadow`. Added a **Shadow / Scars** column to the Party Status Dashboard (between Hope and Endurance) that turns red with a "MISERABLE" tag when `shadow + scars ≥ hopeMax`. The alert also notes the change is local to the Loremaster's copy (no write-back to the player's device). Bumped `sw.js` `CACHE_NAME` → `tor2e-lm-v2`. Verified in preview (10/10 column alignment, correct field write, Miserable flag); synced to `index.html`.
- **2026-06-01 — Fixed invisible Active Combatants list.** `#stance-tracker-card` (Encounter tab) was missing a closing `</div>`, so the **Active Combatants** card (which holds `#active-combatants-list`) was nested inside that `display:none` card. Adversaries added via **+ Add** were rendered into the data/DOM but hidden until players were uploaded (which un-hides the stance tracker). Added the missing `</div>` at `loremaster-tracker.html:500`. Verified in preview: cards now render with no players present; no console errors. Synced to `index.html`.
