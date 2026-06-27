# Copilot Instructions – FantasyDraftLottery

## Project Overview
This is a single-file HTML/CSS/JS Fantasy Draft Lottery app (`index.html`).
All logic, styling, and markup live in that one file.

---

## ✅ Stable Checkpoint — v1.0 (June 2, 2026)

**Everything is working as of this checkpoint.**

- Commit: `63df8add3b19f1d5af4c4098f10aee5e9e383c4f`
- Last meaningful change: Updated title and headers for the 2026 Draft Lottery

### What works at v1.0
- All draft lottery logic functions correctly **for exactly 16 teams**
- UI renders and behaves as expected
- Combo assignment, draw phases, reveal, and final screen all work

### Rollback instructions
If a future change breaks something, reset `main` to this commit:
```
git reset --hard 63df8add3b19f1d5af4c4098f10aee5e9e383c4f
```

---

## ⚠️ Known Limitation (to be fixed)

The current `assignCombos()` fallback for N ≠ 16 slices `NHL_WEIGHTS` (the 16-team percentage array) and rescales only the worst-N entries. This is incorrect and must be replaced with the logic described below.

---

## Planned Improvements

---

### 1. Fix odds for all supported team counts (11–16)

#### Supported sizes
The app must be restricted to **11–16 teams only**. The `num-teams` input should enforce `min="11" max="16"`. Any attempt to enter a value outside this range should be blocked/clamped with a visible validation message.

#### The existing 16-team table is CORRECT and must not be changed
```js
const NHL_COMBOS_16 = [185, 135, 115, 95, 85, 75, 65, 60, 50, 35, 30, 25, 20, 15, 5, 5];
// sum = 1000, worst team first
```

#### New setup toggle: Odds Mode
Add a second toggle in the setup UI, styled identically to the existing "10-Spot Jump Max" toggle:

- **Label:** `NHL-Style Odds`
- **Subtext:** `Top seed stays at 18.5% regardless of field size. Off = proportional scaling across all teams.`
- **Default:** ON (checked)

Store this as `state.nhlStyleOdds` (boolean).

#### NHL-Style Odds mode (toggle ON)
Use hardcoded tables for each supported size. These are derived from official NHL lottery documentation — the top entries are kept identical to the 16-team table, and the final entry absorbs the remainder to reach exactly 1000.

```js
const NHL_COMBO_TABLES = {
  16: [185, 135, 115, 95, 85, 75, 65, 60, 50, 35, 30, 25, 20, 15, 5, 5],
  15: [185, 135, 115, 95, 85, 75, 65, 60, 50, 35, 30, 25, 20, 15, 5],       // last entry = 1000 - sum(first 14) = 5 ✓
  14: [185, 135, 115, 95, 85, 75, 65, 60, 50, 35, 30, 25, 20, 20],          // last entry absorbs remainder → 20
  13: [185, 135, 115, 95, 85, 75, 65, 60, 50, 35, 30, 25, 10],              // last = 1000 - 990 = 10 → adjusted: 10+10=20? recompute: sum(first 12)=950, last=50 → recalc carefully
  12: [185, 135, 115, 95, 85, 75, 65, 60, 50, 35, 30, 70],                  // last = 1000 - 930 = 70
  11: [185, 135, 115, 95, 85, 75, 65, 60, 50, 35, 100],                     // last = 1000 - 900 = 100
};
```

> ⚠️ **Important:** Do NOT use the approximate values above verbatim. Recompute each table yourself by:
> 1. Taking the top (N-1) entries from `NHL_COMBOS_16` in order.
> 2. Computing `remainder = 1000 - sum(first N-1 entries)`.
> 3. Setting the last (best team) entry to `remainder`.
> 4. Verifying the full array sums to exactly 1000.
>
> This is the authoritative algorithm. The values in the code block above are illustrative only.

#### Proportional Odds mode (toggle OFF)
Take all 16 entries from `NHL_COMBOS_16`, compute their sum (1000), then rescale proportionally across N teams:
1. Take the full 16-entry `NHL_COMBOS_16` array.
2. Compute `scaledWeights = NHL_COMBOS_16.map(w => w / 1000)` (these are the 16 proportional weights).
3. For N teams, take `scaledWeights.slice(0, N)` and renormalize so they sum to 1.0: divide each by `scaledWeights.slice(0, N).reduce((a,b)=>a+b, 0)`.
4. Multiply each normalized weight by 1000 and round to nearest integer.
5. Compute `remainder = 1000 - counts.reduce((a,b)=>a+b, 0)` and add it to the last (best team) entry.
6. Verify the array sums to exactly 1000.

In this mode, the worst team gets a higher percentage than 18.5% when N < 16, because the field is smaller.

#### Shared: deriving baseOdds
Regardless of mode, after the combo table is computed:
```js
team.baseOdds = parseFloat(((team.numCombos / 1000) * 100).toFixed(1));
```
Do **not** use the old `NHL_WEIGHTS` percentage array for `baseOdds`.

---

### 2. Per-team combo preview modal

#### Trigger
Add a small **"👁"** icon button inside each `.team-card`, positioned in the bottom-right corner.

Visibility rules:
- Show only when `team.combos` is populated (assignment done for that team).
- Hide once `state.phase === 'final'`.
- The button must not interfere with the existing card click behavior (assign phase).

#### Behaviour
Clicking the button opens a new dedicated overlay (`#combo-preview-overlay`, separate from `#modal-overlay`) that is **read-only** — no draw buttons, no state changes.

The overlay shows:
- A colour bar at the top matching `team.color`
- Team name, original standing, combo count, and odds % in a header
- All combos displayed in a **paginated grid**:
  - Show **50 combos per page** in a compact multi-column layout (e.g. 5 columns)
  - Each combo rendered as `A-B-C-D` in a small pill/badge
  - Pagination controls: `← Prev | Page X of Y | Next →`
- A **"✕ Close"** button at the top-right, and clicking the backdrop also closes
- The overlay is scrollable if needed

Accessible from: end of assignment phase → through draw phases → up to (but not including) `final` phase.

---

### 3. Downloadable PDF report (client-side only)

#### Setup
Add jsPDF via CDN in `<head>`:
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
```

#### Trigger
Add a **"📄 Download Report"** button to `.final-actions` on the final screen. Clicking it calls `generatePDF()`.

#### PDF structure (multi-page)

**Page 1 — Results Summary:**
- Title: league name (from the `<h2>` header text, stripped of emoji)
- Subtitle: "Draft Lottery Results" + date generated
- Rule note: Jump limit on/off, odds mode used
- Final draft order table with columns: Pick # | Team | Original Standing | Movement
  - Movement shown as `↑N`, `↓N`, or `—`
  - Pick 1 and Pick 2 rows visually highlighted (bold or annotated as lottery winners)

**Pages 2–N — Per-team combo listings:**
- Teams ordered worst → best standings (same as sidebar order)
- Each team section header: Team name | Standing | Combos: N | Odds: X.X%
- All combos listed in a grid (6 per row), formatted as `[A-B-C-D]`
- Use `doc.addPage()` when a team's combos overflow the current page
- Add a page number footer to every page

**Download:**
```js
doc.save('draft-lottery-report.pdf');
```

No email, no SMTP, no server — pure browser download via blob URL.

---

## Development Notes for Copilot

- **Single-file architecture**: Keep all changes inside `index.html` unless explicitly asked to split.
- **No build system**: Plain HTML/CSS/JS only. No npm, no bundler, no framework. CDN scripts are fine.
- **Do not touch the 16-team path**: `NHL_COMBOS_16` and the 16-team combo assignment must remain exactly as they are. All new multi-size logic must be additive.
- **Always verify combo counts sum to 1000**: Any combo table generated — NHL-style or proportional — must be asserted to sum to exactly 1000 before use.
- **No silent regressions**: Flag any change that touches `assignCombos()`, `comboKey()`, `comboMap`, or draw matching logic.
- **Preserve state shape**: Do not rename or remove existing fields on the `state` object. Add new fields (`nhlStyleOdds`) additively.
