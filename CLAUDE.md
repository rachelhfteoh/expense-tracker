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

**Calendar** — Month nav (← →, swipe). Summary bar (month total). Sun–Sat grid. Today = dark navy. Selected = light purple. Amounts shown below date. Tap day → panel below with transactions + + button. No FAB (hidden).

**Stats** — Month nav (← →, swipe). SVG donut chart. Category breakdown list (dot, emoji, name, bar, %, amount).

---

## Sheets

- `#add-sheet` — add/edit transaction (date + 🔁 button, amount, category grid, note, save/delete)
- `#repeat-sheet` — repeat picker (stacks above add sheet, z-index 400)
- `#recurring-sheet` — manage recurring rules (stacks above overlay, z-index 400)
- Overlay click: smart — closes topmost sheet only; `subSheet` tracks what's stacked

---

## Styling conventions

- Background: `#f4f4f7` (warm off-white)
- Cards: `#ffffff`, border-radius varies
- Expense color: `#ef4444`
- Today highlight: `#1e293b` (dark navy)
- Selected non-today: `#ede9fe` bg, `#7c3aed` text
- Calendar starts Sunday (col 1). Sun text = red, Sat text = blue.
- Category icon bg: `${cat.color}1a` (10% opacity hex)
- Font: Plus Jakarta Sans (Google Fonts CDN)
- Safe area: `env(safe-area-inset-top)` top, `env(safe-area-inset-bottom)` bottom
