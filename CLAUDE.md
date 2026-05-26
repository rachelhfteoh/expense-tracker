# CLAUDE.md — Expense Tracker

## Session Rules

### Commit message conventions
- Mid-session commits: `"<description> — mid-session checkpoint"`
- End-of-session commits: only use `"end of session"` when user types `/end`

### Before every `/compact`
Update CLAUDE.md and CONTEXT.md, then commit both before compacting.

### At end of session (`/end`)
1. ✅ All changes committed
2. ✅ CONTEXT.md updated
3. ✅ CLAUDE.md updated with any architecture changes

---

## Project files

| File | Purpose |
|---|---|
| `index.html` | Single-file HTML/CSS/JS app |
| `CLAUDE.md` | Rules and architecture |
| `CONTEXT.md` | Current state and next steps |
| `MEMORY.md` | Lessons learned and key decisions |

**To develop:** open `index.html` in Safari. No build tools needed.

---

## App Overview

Personal expense tracker (MYR only). Single `index.html`, no build tools, localStorage, PWA-ready.

**Deployed at:** (not yet deployed — to be set up on GitHub Pages)

---

## Data Model

Persisted to `localStorage` under key `'et_v1'`:

```js
{
  transactions: [{
    id: string,           // uid()
    date: string,         // "YYYY-MM-DD"
    amount: number,       // always positive (expenses only)
    category: string,     // key from CATS array
    note: string,         // free text description
    createdAt: string,    // ISO timestamp
    recurringId: string | null  // links to recurring rule; null if one-off
  }],
  recurring: [{
    id: string,
    amount: number,
    category: string,
    note: string,
    frequency: string,    // see REPEATS array
    startDate: string,    // "YYYY-MM-DD" — first generated date
    lastGenerated: string // last date an instance was generated up to
  }]
}
```

Migration: add backfill in `load()` for any new fields (same pattern as habit tracker).

---

## Architecture

- All state in module-level JS variables
- `render()` rebuilds entire UI on every state change
- `renderContent()` can be called alone for cheaper partial updates (e.g. calendar day tap)
- No build tools, no npm, no bundler

### Key state variables
- `view` — `'transactions' | 'calendar' | 'stats'`
- `txYear, txMon` — month displayed in Transactions tab
- `calYear, calMon` — month displayed in Calendar tab
- `calSel` — ISO date string of selected calendar day
- `stYear, stMon` — month displayed in Stats tab
- `fMode, fEditId, fDate, fAmount, fCat, fNote, fRepeat` — add/edit form state
- `subSheet` — which secondary sheet is stacked above the add sheet (`'repeat' | 'recurring' | null`)
- `calcExpr` — current expression string in the calculator keyboard (e.g. `"3.75+4.95"`)

### Key helpers
- `todayStr()` — current date as "YYYY-MM-DD"
- `uid()` — unique ID
- `rm(n)` — format as "RM X.XX"
- `isoDate(y, m, d)` — build ISO date string
- `parseDate(ds)` — split ISO string into `{ y, m, d }` (m is 0-indexed)
- `addMonths(ds, n)` — add N months, clamping to valid day
- `lastDayOfMonth(y, m)` — ISO date of last day
- `nextOccurrence(ds, freq)` — next date after ds for given frequency
- `generateRecurring()` — called on load; fills missing instances up to today
- `txForMonth(y, m)` — filter transactions by year/month
- `txForDay(ds)` — filter transactions by exact date
- `totalDay(ds)` — sum of expenses for a day
- `totalMonth(y, m)` — sum of expenses for a month
- `evalAmount(raw)` — safely evaluates a math expression string (whitelist regex + Function()); returns number or null
- `resolveAmount(el)` — evaluates input value and updates field (used in saveTx fallback)

### Calculator keyboard helpers
- `openCalc()` — shows `#calc-keyboard`, seeds `calcExpr` from `fAmount`
- `closeCalc()` — hides `#calc-keyboard`
- `updateCalcDisplay()` — syncs `calcExpr` to `#f-amt` field; shows `= X.XX` preview if expression has operator
- `calcInput(ch)` — appends character with guard rules (no double operator, no double decimal)
- `calcBack()` — deletes last character
- `calcEqual()` — evaluates and collapses expression to result
- `calcOK()` — calls `calcEqual()` then `closeCalc()`

---

## Categories (16)

🍎 Groceries · 🍽️ Eating Out · 🚗 Transport · 🛍️ Shopping · 🏠 Housing · 💡 Utilities · 🎬 Entertain · 💊 Health · ✨ Beauty · 📚 Education · 🐾 Pets · 💪 Fitness · 🎁 Gifts · ✈️ Travel · 💼 Work · ❓ Other

---

## Repeat Frequencies (14)

Nothing · Every Day · Weekdays · Weekend · Every Week · Every 2 Weeks · Every 4 Weeks · Every Month · End of Month · Every 2 Months · Every 3 Months · Every 4 Months · Every 6 Months · Annually

- "End of Month" snaps `startDate` to last day of selected month
- `generateRecurring()` runs on every app load; generates missing instances up to today (max 1000 per rule)
- Editing a recurring instance only changes that instance — rule is unchanged
- Delete a rule via "🔁 Recurring" button in Transactions header

---

## Views

**Transactions (default)** — Month nav (← →, swipe). Summary bar (Expenses, count). Transactions grouped by date newest first. FAB + to add. Tap row → edit sheet.

**Calendar** — Month nav (← →, swipe). Summary bar (month total). Sun–Sat grid. Today = purple gradient. Selected = light purple. Amounts shown below date. Tap day → panel below with transactions + + button. No FAB (hidden).

**Stats** — Month nav (← →, swipe). SVG donut chart. Category breakdown list (dot, emoji, name, bar, %, amount).

---

## Sheets & Overlays

- `#add-sheet` — add/edit transaction (date + 🔁 button, amount, category grid, note, save/delete)
- `#repeat-sheet` — repeat picker (stacks above add sheet, z-index 400)
- `#recurring-sheet` — manage recurring rules (stacks above overlay, z-index 400)
- `#calc-keyboard` — custom calculator keyboard (z-index 500, fixed at bottom, slides up on amount tap)
- Overlay click: smart — checks calc first, then topmost sheet; `subSheet` tracks what's stacked

### Z-index stack
| Layer | z-index |
|---|---|
| Overlay | 200 |
| Add sheet | 300 |
| Repeat / Recurring sheet | 400 |
| Calculator keyboard | 500 |
| Toast | 600 |

---

## Styling conventions

### Colourful theme (applied Session 2)
The app matches the Habit Tracker aesthetic. Theme overrides are added as a CSS block near the bottom of `<style>` — original variables remain for fallback.

- **Background:** `linear-gradient(150deg, #fdf4ff 0%, #eef2ff 45%, #f0fdfa 100%)` — purple/lavender to mint
- **Header / Nav:** `rgba(255,255,255,0.88)` + `backdrop-filter: blur(20px)` (frosted glass)
- **Header title:** gradient text `linear-gradient(135deg, #7c3aed 0%, #ec4899 100%)`
- **Primary UI accent:** `#7c3aed` (violet/purple) — FAB, nav active, save button, today dot, calc operators
- **Expense amounts:** `#ef4444` red (data colour — unchanged)
- **Transaction cards:** `rgba(255,255,255,0.85)` glass, border-radius 16px, margin 0 12px 10px
- **Summary stat cards:** pink gradient (expenses), purple gradient (count)
- **Today highlight:** purple gradient (was dark navy)
- **Selected non-today:** `#ede9fe` bg, `#7c3aed` text (unchanged)

### Base variables (still in `:root`)
- `--bg: #f4f4f7` (fallback only — body uses gradient)
- `--card: #ffffff`
- `--expense: #ef4444`
- `--today-bg: #1e293b` (overridden by theme)
- Font: Plus Jakarta Sans (Google Fonts CDN)
- Safe area: `env(safe-area-inset-top)` top, `env(safe-area-inset-bottom)` bottom

### Amount input
- `type="text" inputmode="none" readonly onclick="openCalc()"` — suppresses native keyboard, opens calculator
- Do NOT use `type="number"` — prevents expression strings like `3.75+4.95`
