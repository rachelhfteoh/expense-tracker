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
    category: string,     // key from CATS or customCats
    note: string,         // short label
    description: string,  // long-form notes
    photo: string | null, // base64 JPEG data URL, max 800px
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
  }],
  customCats: [{
    key: string,          // "custom_<uid>"
    label: string,
    emoji: string,        // auto-assigned '📦'
    color: string         // auto-assigned from CAT_COLORS palette
  }],
  hiddenCats: [string]    // keys of built-in cats the user has hidden
}
```

Migration: add backfill in `load()` for any new fields (same pattern as habit tracker).

---

## Architecture

- All state in module-level JS variables
- `render()` rebuilds entire UI on every state change
- `renderContent()` can be called alone for cheaper partial updates (e.g. calendar day tap, category pick)
- No build tools, no npm, no bundler

### Key state variables
- `view` — `'transactions' | 'calendar' | 'stats' | 'add'`
- `addViewPrev` — view to return to when closing the add page (e.g. `'transactions'`)
- `txYear, txMon` — month displayed in Transactions tab
- `txWeekOffset` — week offset for the week strip (0 = current week, negative = past weeks)
- `txSelDay` — ISO date of selected day in the week strip (default: today)
- `calYear, calMon` — month displayed in Calendar tab
- `calSel` — ISO date string of selected calendar day
- `stYear, stMon` — month displayed in Stats tab
- `fMode, fEditId, fDate, fAmount, fCat, fNote, fRepeat` — add/edit form state
- `fDescription` — long-form description field state
- `fPhoto` — base64 photo data URL (or null)
- `subSheet` — which secondary sheet is open (`'repeat' | 'recurring' | 'cat' | null`)
- `calcExpr` — current expression string in the calculator keyboard (e.g. `"3.75+4.95"`)
- `activePanel` — which panel is visible in the bottom panel area (`'calc' | 'cat' | null`)
- `catAddMode` — whether the "New Category" name-entry form is open in `#cat-sheet`
- `newCatLabel` — new category name being typed

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
- `getAllCats()` — returns `[...CATS, ...data.customCats]` filtered by `data.hiddenCats`
- `getCat(key)` — looks up cat in CATS then customCats, falls back to 'other'
- `syncFormState()` — reads DOM inputs into fDate/fAmount/fNote/fDescription state vars (call before opening any sub-sheet)
- `compressImage(dataUrl, cb)` — resizes to max 800px, JPEG 70%, returns via callback
- `closeAddView()` — closes sub-sheets, resets overlay, sets view=addViewPrev, calls render()

### Bottom panel helpers (Session 4)
- `openCalc()` — sets `activePanel='calc'`, shows `#calc-inner`, hides `#cat-inner`, calls `updateCalcDisplay()`
- `closeCalc()` — sets `activePanel=null`, hides `#calc-inner`
- `openCatPanel()` — sets `activePanel='cat'`, hides calc, calls `renderCatInner()`, shows `#cat-inner`
- `renderCatInner()` — builds category grid HTML into `#cat-inner` (excludes 'other'; adds "Add" tile; shows ✕ only if category has no tagged transactions)
- `pickCatInline(key)` — sets `fCat`, snaps panel back to calc, calls `renderContent()`
- `deleteCatInline(key)` — custom cats: removed from `customCats`; built-in cats: added to `hiddenCats`; calls `renderCatInner()`
- `initPanel()` — called by `renderContent()` after DOM rebuild; restores panel to `activePanel` state
- `updateCalcDisplay()` — syncs `calcExpr` to `#f-amt`; shows `= X.XX` preview if expression has operator
- `calcInput(ch)` — appends character; clears `0.00` default on first digit
- `calcBack()` — deletes last character; clears `0.00` in one press
- `calcEqual()` — evaluates and formats result to 2dp
- `calcOK()` — calls `calcEqual()` then `closeCalc()`

### Category "New Category" sheet helpers (Session 4)
- `openCatNew()` — sets `catAddMode=true`, renders name form into `#cat-sheet`, shows sheet + overlay
- `closeCatNew()` — closes sheet, calls `openCatPanel()` to return to inline cat grid
- `renderCatSheet()` — now ONLY renders the name-entry form inside `#cat-sheet` (no longer renders the grid)
- `saveCustomCat()` — saves new cat (emoji='📦', color auto from palette), closes sheet, calls `renderCatInner()`

### Week strip helpers (Session 5)
- `getTxWeekDays(offset)` — returns 7 ISO dates for the week at offset (0 = current), starting Sunday
- `renderTxWeekStrip()` — builds strip HTML into `#tx-week-strip`; called by `renderContent()` after transactions HTML set
- `selectTxDay(ds)` — sets `txSelDay`; syncs `txYear/txMon` if different month; scrolls to `#tx-day-{ds}`
- `changeTxWeek(dir)` — increments `txWeekOffset`; syncs month if needed
- `initTxWeekSwipe()` — attaches touch/mouse swipe handlers to the strip (idempotent via `_swipeInit` flag)

---

## Categories (15 selectable + custom + hidden Other)

🍎 Groceries · 🍽️ Eating Out · 🚗 Transport · 🛍️ Shopping · 🏠 Housing · 💡 Utilities · 🎬 Entertain · 💊 Health · ✨ Beauty · 📚 Education · 🐾 Pets · 💪 Fitness · 🎁 Gifts · ✈️ Travel · 💼 Work · _(❓ Other — hidden from picker, used as code fallback only)_

Custom categories: `key: 'custom_<uid>'`, `emoji: '📦'` (auto), color cycles through `CAT_COLORS[]`. The "Add" tile in the inline category panel opens `#cat-sheet` for name entry only. `getCat()` falls back to 'other' if key not found.

**Default category** when opening add view: `'groceries'` (not 'other').

**Delete rules:** ✕ button only shown if `data.transactions` has no entry with that category key. Custom cats are removed from `customCats`; built-in cats are added to `hiddenCats`. `getAllCats()` filters out `hiddenCats`.

---

## Repeat Frequencies (14)

Nothing · Every Day · Weekdays · Weekend · Every Week · Every 2 Weeks · Every 4 Weeks · Every Month · End of Month · Every 2 Months · Every 3 Months · Every 4 Months · Every 6 Months · Annually

- "End of Month" snaps `startDate` to last day of selected month
- `generateRecurring()` runs on every app load; generates missing instances up to today (max 1000 per rule)
- Editing a recurring instance only changes that instance — rule is unchanged
- Delete a rule via "🔁 Recurring" button in Transactions header

---

## Views

**Transactions (default)** — Month nav (← →, swipe). Week strip (S M T W T F S, swipeable, expense dots, tapping scrolls to day group). Summary bar (Expenses, count). Transactions grouped by date, sorted highest amount first. FAB + to add. Tap row → add view.

**Calendar** — Month nav (← →, swipe). Summary bar (month total). Sun–Sat grid. Today = purple gradient. Selected = light purple. Amounts shown below date. Tap day → panel below with transactions sorted highest first + + button. No FAB (hidden).

**Stats** — Month nav (← →, swipe). SVG donut chart. Category breakdown list (dot, emoji, name, bar, %, amount).

**Add/Edit (`view = 'add'`)** — Full-page view. Header: back button (← Trans./Calendar/Stats) + title (centred, invisible spacer on right). Nav bar and FAB hidden. Form rows: fixed 44px height each. Fields: Date + 🔁 icon-only repeat button, Amount (tap → calc panel), Category (tap → cat panel), Note, Description (min-height 90px) + camera. Bottom panel switches between calc and category grid. Fixed bottom action bar: Save (left, purple) + Delete (right, red outlined) — always visible; Delete on new expense = discard.

### Transaction row layout
```
[icon] Category    Note text         RM X.XX
```
- `.tx-left` (120px, row): icon circle (42px) + category label (12px grey)
- `.tx-info` (flex:1): note text (14px bold); falls back to `—` if no note
- `.tx-right`: amount in red

---

## Sheets & Overlays

- `#add-sheet` — present in HTML but unused (kept to avoid null refs in closeAllSheets)
- `#repeat-sheet` — repeat picker (slides over add view, z-index 400, shows overlay)
- `#recurring-sheet` — manage recurring rules (sheet from Transactions header, z-index 400)
- `#cat-sheet` — ONLY used for "New Category" name-entry form (z-index 400, shows overlay); category grid is now inline in `#cat-inner`
- `#bottom-panel` — inline div at bottom of add form; contains `#calc-inner` (calculator) and `#cat-inner` (category grid); one shown at a time
- `#photo-action` — Camera/Gallery action sheet (z-index 550, custom iOS-style)
- `.add-action-bar` — fixed bottom bar (z-index 50); Save + Delete buttons
- Overlay click: handles repeat/recurring/cat sheets only; inline panels do NOT use the overlay

### Z-index stack
| Layer | z-index |
|---|---|
| Add action bar | 50 |
| Overlay | 200 |
| Add sheet (unused) | 300 |
| Repeat / Cat / Recurring sheet | 400 |
| Photo action sheet | 550 |
| Toast | 600 |

---

## Styling conventions

### Colourful theme (applied Session 2)
The app matches the Habit Tracker aesthetic. Theme overrides are added as a CSS block near the bottom of `<style>` — original variables remain for fallback.

- **Background:** `linear-gradient(150deg, #fdf4ff 0%, #eef2ff 45%, #f0fdfa 100%)` — purple/lavender to mint
- **Header / Nav:** `rgba(255,255,255,0.88)` + `backdrop-filter: blur(20px)` (frosted glass)
- **Header title (main views):** gradient text `linear-gradient(135deg, #7c3aed 0%, #ec4899 100%)`
- **Add-page header title:** plain `#111827` (no gradient) — use `.add-page-header-title` class
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

### Form rows (Session 5)
- All `.form-row` elements: fixed `height: 44px`, `padding: 0 20px`, `box-sizing: border-box`
- `.amount-input`: `font-size: 15px; font-weight: 700` (slightly larger than other fields but not huge)
- `.desc-textarea`: `min-height: 90px` for memo typing
- Repeat button: icon-only (🔁), `width: 34px; height: 34px`, no label text

### Amount input
- `type="text" inputmode="none" readonly onclick="openCalc()"` — suppresses native keyboard, opens calculator
- Do NOT use `type="number"` — prevents expression strings like `3.75+4.95`
- Defaults to `'0.00'`; cleared on first digit; always formatted to 2dp after `=` or OK

### Bottom panel layout
- `#bottom-panel` sits inline in `buildAddPage()` below the form fields; `margin-top: 12px`
- `#calc-inner`: `background: #f3f4f6`, `border-radius: 16px 16px 0 0`, `display:none` by default
- `#cat-inner`: `background: #fff`, `border-radius: 16px 16px 0 0`, `max-height: 280px`, scrollable
- Toggle via `openCalc()` / `openCatPanel()` — direct DOM show/hide, no re-render needed
- `initPanel()` is called after every `renderContent()` to restore state after DOM rebuild

### Category grid (inline panel)
- 4 columns (`cat-grid`, no `cat-grid-3` override)
- `.cat-item`: `padding: 8px 4px`, `gap: 3px`
- `.cat-emo`: `font-size: 20px`
- `.cat-nm`: `font-size: 10px`
- ✕ delete button only shown when `!usedKeys.has(c.key)` (no tagged transactions)
