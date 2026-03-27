# NSRT → MRT Converter — Claude Code Instructions

## Project Overview

A single-file browser tool that converts **Northern Sky Raid Tools (NSRT)** cooldown exports into **Method Raid Tools (MRT)** note format for World of Warcraft raid leaders.

No build system. No framework. No dependencies. Pure HTML/CSS/JS in one file: `index.html`.

Deployed via **GitHub Pages** from the `main` branch (root `/`).

---

## Repo Structure

```
nsrt-mrt/
├── index.html        ← The entire app. Single source of truth.
├── CLAUDE.md         ← This file
├── README.md         ← GitHub readme
└── .gitignore
```

Do not introduce a build step, bundler, npm, or framework unless explicitly asked. This is intentionally a zero-dependency project.

---

## Format Reference

### NSRT Input Format

```
EncounterID:3180;Difficulty:Heroic;Name:Lightblinded Vanguard
time:13;ph:1;bossSpell:1246749;tag:Deathtrolls;spellid:391528;
time:31;ph:1;tag:Apöllô;spellid:64843;
```

- Fields are semicolon-delimited key:value pairs
- Header line always contains `EncounterID:`
- CD lines always contain both `time:` and `spellid:`
- `bossSpell:` is optional — marks which boss ability triggers the CD
- `ph:` (phase) is present but not used in MRT; discard it
- `tag:` is the player name
- `Difficulty` can be a string (`Heroic`, `Normal`, `Mythic`, `LFR`) or already a number

### MRT Output Format

```
EncounterID:3180 Difficulty:15 Name:Lightblinded Vanguard
{time:13} {bossAbility:1246749} {spell:391528} Deathtrolls
{time:31} {spell:64843} Apöllô
```

- Header fields are space-delimited (not semicolons)
- Difficulty must be numeric: LFR=17, Normal=14, Heroic=15, Mythic=16
- `{time:X}` — encounter timer anchor
- `{spell:X}` — player spell icon
- `{bossAbility:X}` — optional boss spell icon (user-toggleable)
- Player name is plain text, no braces
- `ph:` is dropped entirely

---

## Core Parser Logic

Located in the `<script>` block of `index.html`.

### Key functions

| Function | Purpose |
|---|---|
| `parseLine(line)` | Splits a semicolon-delimited NSRT line into a `{key: value}` object. Keys are lowercased. |
| `resolveDifficulty(raw, override)` | Maps string difficulty → numeric. Falls back to `15` (Heroic) if unknown. |
| `convert()` | Main entry point. Reads textarea, processes line by line, writes output. |
| `renderStats(cds, boss, errs, total)` | Updates the stats bar below the panels. |
| `showError(msg)` | Displays the red error box. |
| `copyOutput()` | Clipboard copy with visual feedback on the button. |
| `clearInput()` | Resets both panels and clears stats/errors. |
| `loadSample()` | Loads the hardcoded Lightblinded Vanguard sample and runs convert(). |

### Conversion logic (line-by-line)

1. If line matches `/EncounterID:/i` → it's a header. Parse fields, emit `EncounterID:X Difficulty:Y Name:Z`
2. If line matches both `/time:/i` and `/spellid:/i` → it's a CD assignment. Build the MRT token string.
3. Anything else → pass through as `-- comment` and add a warning.

### User options (checkboxes/selects in the controls bar)

| Option | ID | Effect |
|---|---|---|
| Difficulty override | `#diffOverride` | Forces a specific numeric difficulty instead of auto-detecting |
| Include `{bossAbility:X}` | `#keepBossAbility` | Toggle whether bossSpell tokens are emitted |
| Name before spell | `#nameFirst` | Swaps output order to `PlayerName {spell:X}` vs `{spell:X} PlayerName` |

---

## Design System

The visual aesthetic is **WoW-adjacent dark UI**: near-black backgrounds, gold accents, monospace output. Maintain this when adding UI.

### CSS variables (defined in `:root`)

| Variable | Use |
|---|---|
| `--bg` | Page background (`#0d0f14`) |
| `--surface` | Header / card surface (`#161920`) |
| `--surface2` | Input backgrounds, buttons (`#1e222c`) |
| `--gold` | Primary accent, output text color (`#c8a84b`) |
| `--gold-dim` | Border accent (`#8a6e28`) |
| `--gold-glow` | Transparent gold fills |
| `--text` | Primary text (`#e8e2d4`) |
| `--text-dim` | Secondary/label text |
| `--text-muted` | Placeholder / disabled |
| `--green` | Success / output dot |
| `--blue` | Input dot |
| `--border` / `--border2` | Subtle / emphasis borders |

Never hardcode colors outside `:root`. Always use variables for new UI elements.

### Component patterns

- **Buttons**: `.btn` base class. `.btn.primary` for the main convert action. Never use `<form>` tags.
- **Panels**: `.panel` > `.panel-header` > `.panel-label` + `.panel-actions` > `textarea`
- **Controls bar**: `.controls` > `.controls-left` (options) + primary button (right-aligned)
- **Stats bar**: `#statsBar` — flex row of `.stat` items with `.sep` dividers
- **Error box**: `#errorBox` — hidden by default, shown via `style.display = 'block'`

---

## Adding Features

When adding a new option to the controls bar:

1. Add the control HTML inside `.controls-left`
2. Read its value inside `convert()` before the loop
3. Apply it in the line processing logic
4. Update `renderStats()` if the new option produces a trackable count

When adding a new output token type:

1. Document the NSRT input field and the MRT output token in this file under **Format Reference**
2. Add detection in `convert()` — keep the if/else chain ordered: header check first, CD check second, fallback last
3. Update the sample data if the new token type isn't represented

---

## GitHub & Deployment

### Repo

- Remote: `https://github.com/[your-username]/nsrt-mrt`
- Default branch: `main`
- GitHub Pages source: **root of `main` branch** — `index.html` is served at the root

### Deploy steps (first time)

```bash
git init
git add .
git commit -m "init: NSRT to MRT converter"
git remote add origin https://github.com/[your-username]/nsrt-mrt.git
git push -u origin main
# Then enable GitHub Pages in repo Settings → Pages → Deploy from branch → main / (root)
```

### Routine update workflow

```bash
git add index.html
git commit -m "feat: [short description]"
git push
```

GitHub Pages auto-deploys on push to `main`. No CI needed.

### Commit message conventions

```
feat: add [feature name]
fix: [what was broken]
refactor: [what changed internally]
chore: update readme / docs
```

---

## What NOT to Do

- Do not add `npm`, `package.json`, or any build tooling unless explicitly asked
- Do not split into multiple files (CSS, JS) — keep it single-file
- Do not add a framework (React, Vue, etc.) unless explicitly asked
- Do not change the dark gold aesthetic without being asked
- Do not alter `parseLine()` or `resolveDifficulty()` without updating the Format Reference section above
- Do not add `<!-- HTML comments -->` or `/* CSS comments */` to `index.html` — they waste space in a single-file app

---

## Known Limitations / Future Work

- Multi-phase support: NSRT exports can have multiple `ph:` values per boss (e.g. phase 2 CDs). Currently `ph:` is discarded. A future enhancement could group output by phase with `-- Phase 2` section headers.
- MRT `{remark:}` token support: MRT supports inline text remarks; not currently emitted.
- Batch mode: convert multiple boss exports at once (stacked header blocks in one paste).
- Dark/light theme toggle.
