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

### Testing — always commit and push before asking Rachel to test
Rachel tests exclusively on GitHub Pages (https://rachelhfteoh.github.io/expense-tracker/).
Every change MUST be committed and pushed before asking her to test.
Never ask her to test without pushing first — she will always see the old cached version.

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

**Deployed at:** https://rachelhfteoh.github.io/expense-tracker/

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
- `view` — `'transactions' | 'calendar' | 'stats' | 'add' | 'cat-detail' | 'recurring' | 'monthly'`
- `addViewPrev` — view to return to when closing the add page (e.g. `'transactions'`, `'cat-detail'`)
- `txYear, txMon` — month displayed in Transactions tab
- `txWeekOffset` — week offset for the week strip (0 = current week, negative = past weeks)
- `txSelDay` — ISO date of selected day in the week strip (default: today); Transactions tab shows only this day's transactions
- `calYear, calMon` — month displayed in Calendar tab
- `calSel` — ISO date string of selected calendar day
- `stYear, stMon` — month displayed in Stats tab
- `catDetailKey` — category key currently shown in cat-detail view
- `fMode, fEditId, fDate, fAmount, fCat, fNote, fRepeat` — add/edit form state
- `fDescription` — long-form description field state
- `fPhoto` — base64 photo data URL (or null)
- `fIsRecurring` — true when editing a transaction that has/had a `recurringId`; used to show 🔁 in Edit Expense header AND to hide/show the 🔁 repeat button in the date row
- `subSheet` — which secondary sheet is open (`'repeat' | 'recurring' | 'cat' | 'monthly-filter' | null`)
- `ruleEditId` — ID of the recurring rule being edited in the Edit Rule sheet (null when closed)
- `ruleEditNote` — note field state for Edit Rule sheet
- `ruleEditCents` — amount in integer cents for Edit Rule sheet numpad (same cash register style as main calc)
- `ruleEditFreq` — frequency field state for Edit Rule sheet
- `ruleEditCat` — category field state for Edit Rule sheet (currently unused — category is read-only in sheet)
- `calcCents` — current right operand in integer cents (e.g. `1234` = RM 12.34)
- `calcLeftCents` — left operand in cents when an operator is pending (null otherwise)
- `calcOp` — pending operator: `'+'|'-'|'*'|'/'|null`
- `activePanel` — which panel is visible in the bottom panel area (`'calc' | 'cat' | null`)
- `catAddMode` — whether the "New Category" name-entry form is open in `#cat-sheet`
- `newCatLabel` — new category name being typed
- `monthlyFilterCat` — category key currently filtering the Monthly tab (`''` = all categories)
- `collapsedYears` — `Set` of year strings currently collapsed in the Monthly tab year list

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
- `syncFormState()` — reads DOM inputs/spans into fDate/fAmount/fNote/fDescription state vars (call before opening any sub-sheet)
- `fmtFormDate(ds)` — formats ISO date as "27 May 2026" for display in the date row span
- `compressImage(dataUrl, cb)` — resizes to max 800px, JPEG 70%, returns via callback
- `closeAddView()` — closes sub-sheets, resets overlay, sets view=addViewPrev, calls render()
- `goToMonth(y, m)` — sets txYear/txMon, calls syncTxToMonth(), switches to transactions view
- `exportData()` — serialises data to JSON, triggers browser download as `expenses-backup.json`
- `importData(input)` — reads selected .json file, validates, confirms, replaces all data, saves, re-renders
- `openDataSheet()` / `closeDataSheet()` — shows/hides `#data-action` action sheet

### Bottom panel helpers (Session 4)
- `openCalc()` — sets `activePanel='calc'`, inits `calcCents` from `fAmount`, shows `#calc-inner`, calls `updateCalcDisplay()`
- `closeCalc()` — sets `activePanel=null`, hides `#calc-inner`
- `openCatPanel()` — sets `activePanel='cat'`, hides calc, calls `renderCatInner()`, shows `#cat-inner`
- `renderCatInner()` — builds category grid HTML into `#cat-inner` (excludes 'other'; adds "Add" tile; shows ✕ only if category has no tagged transactions)
- `pickCatInline(key)` — sets `fCat`, sets `activePanel=null` (closes panel), calls `renderContent()`
- `deleteCatInline(key)` — custom cats: removed from `customCats`; built-in cats: added to `hiddenCats`; calls `renderCatInner()`
- `initPanel()` — called by `renderContent()` after DOM rebuild; restores panel to `activePanel` state
- `updateCalcDisplay()` — formats `calcCents` via `centsToStr()` into `#f-amt`; shows operator preview; syncs `fAmount`
- `calcInput(ch)` — digit: shifts `calcCents` left (`*10 + digit`); `'00'`: shifts two places; operator: commits left operand, resets right to 0
- `calcBack()` — removes last digit (`floor(calcCents/10)`); if right=0 and op pending, cancels the operator
- `calcEqual()` — applies pending operator via `applyOpCents()`
- `calcOK()` — calls `calcEqual()` then `closeCalc()`
- `centsToStr(c)` — formats integer cents as "X,XXX.XX" string (no RM prefix)
- `applyOpCents(left, op, right)` — applies operator on cent values, returns result in cents (min 0)

### Category "New Category" sheet helpers (Session 4)
- `openCatNew()` — sets `catAddMode=true`, renders name form into `#cat-sheet`, shows sheet + overlay
- `closeCatNew()` — closes sheet, calls `openCatPanel()` to return to inline cat grid
- `renderCatSheet()` — now ONLY renders the name-entry form inside `#cat-sheet` (no longer renders the grid)
- `saveCustomCat()` — saves new cat (emoji='⭐', color auto from palette), closes sheet, calls `renderCatInner()`

### Monthly tab filter helpers (Session 15)
- `openMonthlyCatPicker()` — builds category list (only cats with transactions) into `#monthly-filter-body`, sets `subSheet='monthly-filter'`, shows sheet + overlay
- `pickMonthlyCat(key)` — sets `monthlyFilterCat`, closes picker, calls `renderContent()`
- `closeMonthlyCatPicker()` — hides sheet + overlay, resets `subSheet`
- `buildMonthly()` filters `data.transactions` by `monthlyFilterCat` before computing `monthMap`; empty string = all transactions (default behaviour unchanged)

### Monthly tab year collapse helpers (Session 16)
- `toggleYearCollapse(yr)` — adds/removes year string from `collapsedYears`, calls `renderContent()`
- `collapseAllYears()` — adds all years from `data.transactions` to `collapsedYears`, calls `renderContent()`
- `expandAllYears()` — clears `collapsedYears`, calls `renderContent()`
- Year/Month toggle buttons sit in the pill row (right side); Year button active when `collapsedYears.size > 0`

### Month navigation helpers (Session 8)
- `weekOffsetForDate(ds)` — returns the integer week offset from the current week to the week containing `ds` (negative = past)
- `syncTxToMonth()` — sets `txSelDay` to last day of `txYear/txMon` (capped at today), then sets `txWeekOffset` via `weekOffsetForDate`; called by `prevTx()`/`nextTx()` to keep strip in sync
- `nextTx()` / `nextCal()` / `nextSt()` — blocked when already at current month; `>` button dimmed to 30% opacity

### Week strip helpers (Session 5)
- `getTxWeekDays(offset)` — returns 7 ISO dates for the week at offset (0 = current), starting Sunday
- `renderTxWeekStrip()` — builds strip HTML into `#tx-week-strip`; called by `renderContent()` after transactions HTML set
- `selectTxDay(ds)` — sets `txSelDay`, syncs `txYear/txMon`, calls `renderContent()` — shows only that day's transactions
- `changeTxWeek(dir)` — increments `txWeekOffset`; syncs month; calls `renderContent()`
- `initTxWeekSwipe()` — attaches touch/mouse swipe handlers to the strip (idempotent via `_swipeInit` flag)

### Swipe-to-delete helpers (Session 10)
- `swipeDeleteTx(id)` — removes transaction by id, saves, calls `renderContent()`, shows "Deleted" toast
- `initSwipeRows()` — attaches touch swipe handlers to all `.tx-row-wrap` elements (idempotent via `_swipeInit` flag); called by `renderContent()` after `renderTxWeekStrip()`
- `closeOpenSwipeRow()` — snaps the currently open swipe row back to closed; called by `switchView()` and when a second row is swiped
- `_openWrap` — module-level variable tracking which `.tx-row-wrap` is currently open (or null)
- Swipe threshold: 38px (half of `SWIPE_W = 76px`) to commit open/close
- Tap on open row: captured in the capture phase to close without firing `openEdit()`

### Category detail helpers (Session 6)
- `openCatDetail(key)` — sets `catDetailKey`, `view='cat-detail'`, calls `render()`
- `closeCatDetail()` — sets `view='stats'`, calls `render()` (back button label shows "Categories")
- `buildCatDetail()` — renders transactions for `catDetailKey` in `stYear/stMon`; grouped by date, uses `catDetailRowHtml()`
- `catDetailRowHtml(t)` — compact row: note + amount only (no icon/category label since category shown in header)

---

## Categories (16 selectable + custom + hidden Other)

🍎 Groceries · 🍽️ Eating Out · 🚗 Transport · 💡 Utilities · 💊 Health · 🌸 Beauty · 💪 Fitness · 🦞 Seafood · 🍿 Snacks · 🥦 Veggies · 🍇 Fruits · 🥐 Pastries · 🏠 Household · 🔔 Subscriptions · 🛡️ Insurance · 🔮 Crystal · _(❓ Other — hidden from picker, used as code fallback only)_

Custom categories: `key: 'custom_<uid>'`, `emoji: '⭐'` (auto), color cycles through `CAT_COLORS[]`. The "Add" tile in the inline category panel opens `#cat-sheet` for name entry only. `getCat()` falls back to 'other' if key not found.

**Default category** when opening add view: `''` (empty). Category row shows "Pick a category" placeholder. `saveTx()` shows toast and blocks save if `fCat` is empty.

**Delete rules:** ✕ button only shown if `data.transactions` has no entry with that category key. Custom cats are removed from `customCats`; built-in cats are added to `hiddenCats`. `getAllCats()` filters out `hiddenCats`. If deleted cat was selected (`fCat === key`), resets to `''`.

---

## Repeat Frequencies (2)

Nothing · Every Month

- Simplified to 2 options only — quarterly/bi-annual/annual removed as not needed
- `generateRecurring()` runs on every app load; generates missing instances up to today (max 1000 per rule)
- Editing a recurring transaction ALSO updates the rule's `note`, `amount`, `category` — keeps Recurring tab in sync
- 🔁 badge in transaction rows: shown only while the rule still exists (`data.recurring.some(r => r.id === t.recurringId)`)
- 🔁 icon in Edit Expense header: shown whenever `fIsRecurring` is true — permanent marker even after rule deleted
- `recurringId` is NOT nulled out when a rule is deleted — kept for the Edit header indicator
- 🔁 repeat button in date row: shown when `fMode === 'add' || !fIsRecurring` — hidden for existing recurring transactions
- Adding a repeat frequency while editing a non-recurring transaction creates a NEW rule and links it via `recurringId`
- **Deleting a recurring transaction** (swipe or Delete button): always removes the specific transaction first, then removes the rule + all strictly-future instances (`date > today`). No second prompt.
- **Edit Rule sheet** (`#recurring-sheet`): tap a rule row in Recurring tab → opens sheet. Fields: Since (read-only), Category (read-only), Note (editable), Amount (cash register numpad, same as main calc), Frequency picker. Save syncs note/amount to today+future transactions; frequency change also deletes future instances and regenerates.
- `deleteRuleFromView(id)` — deletes rule + all instances `date >= today` (today inclusive). Confirm prompt: "Delete this and all future recurring transactions?"

---

## Views

**Transactions (default)** — Month nav (← →, swipe). Future months blocked (> dims to 30%). Week strip (S M T W T F S, swipeable, expense dots). Day header always shows: date + day tag + inline purple `+` button (right). Summary bar (day expenses, count). FAB hidden in this view. Tap row → edit view.

**Calendar** — Month nav (← →, swipe). Future months blocked. Summary bar (month total). Sun–Sat grid. Today = purple gradient. Selected = light purple. Amounts shown below date. Tap day → panel below with transactions sorted highest first + + button. FAB hidden.

**Categories (`view = 'stats'`)** — Month nav (← →, swipe). Future months blocked. FAB hidden. No donut chart — replaced with a single summary bar showing Total Expenses for the month. Category breakdown list below (emoji, name, %, amount). `stats-pct`: fixed width 44px, right-aligned. `stats-amt`: fixed width 110px, right-aligned. Tap a category row → cat-detail view. Nav label and header title show "Categories" (internal view key remains `'stats'`).

**Add/Edit (`view = 'add'`)** — Full-page view. Header: back button (← Trans./Calendar/Stats/Detail) + title (centred, invisible spacer on right). Nav bar and FAB hidden. Form rows: fixed 44px height each. Fields: Date + 🔁 icon-only repeat button, Amount (tap → calc panel), Category (tap → cat panel), Note, Description (min-height 90px) + camera. Bottom panel switches between calc and category grid. Fixed bottom action bar: Save (left, purple) + Delete (right, red outlined) — always visible; Delete on new expense = discard.

**Category detail (`view = 'cat-detail'`)** — Full-page. Header: `← Stats` + `[emoji] Category` + month subtitle. Nav/FAB hidden. Summary bar (total, count). Transactions for that category in `stYear/stMon`, grouped by date, compact rows (note + amount only). Tap row → edit (returns to cat-detail after save).

**Recurring (`view = 'recurring'`)** — 4th nav tab. Header: title only (no month nav). FAB hidden. Summary bar (active rule count + monthly commitment total). Glass card list of all `data.recurring` rules: emoji, note, sub-line shows `category · frequency · amount`. Tap row → opens Edit Rule sheet (`#recurring-sheet`). ✕ button → `confirm()` → deletes rule + all instances `>= today`; past entries preserved. `buildRecurring()` / `deleteRuleFromView(id)` / `openRuleEdit(id)`.

**Monthly (`view = 'monthly'`)** — 5th nav tab. Header: title + settings gear ⚙ (top right). FAB hidden. Top row: filter pill (left) + Year/Month toggle buttons (right). No filter: no summary bar, chart and list show all transactions. Filtered: summary bar shows Peak Month + Peak Amount; chart and list show that category only; peak month row gets a "Peak" purple badge. SVG line chart (up to last 12 months, smooth cardinal spline, filled gradient area, dots — peak dot larger). Month list grouped by year (newest first); year header shows year label (left) + year total in purple (right); tap header to collapse/expand that year's months. Month rows: month name + amount only (no bar graph). Year/Month toggle: Year button collapses all years, Month button expands all. Settings gear opens `#data-action` sheet with Export / Import options.

### Transaction row layout
```
[icon] Category    Note text         RM X.XX
```
- Each row is wrapped: `.tx-row-wrap` (overflow:hidden, position:relative) → `.tx-swipe-del` button (76px wide, absolute right) + `.tx-row`
- `.tx-row` has `background: var(--card)` — required so it visually covers the red delete button when not swiped
- `.tx-row-wrap:last-child .tx-row { border-bottom: none }` — use this, NOT `.tx-row:last-child` (which now incorrectly matches every row since each is the only `.tx-row` in its wrap)
- `.tx-left` (120px, row): icon circle (32px) + category label (12px, `#6b7280`)
- `.tx-info` (flex:1): note text (12px bold); falls back to `—` if no note
- `.tx-right`: amount in black (12px bold) — red removed as all entries are expenses

### Cat-detail row layout (`.cd-row`)
```
Note text                             RM X.XX
```
- Compact: `padding: 10px 16px`, no icon, no category label
- `.cd-note`: 12px/500, flex:1; `.tx-amount` overridden to 12px within `.cd-row`

---

## Sheets & Overlays

- `#data-action` — Export/Import action sheet (z-index 550, same style as #photo-action); opened by ⚙ in Monthly header
- `#add-sheet` — present in HTML but unused (kept to avoid null refs in closeAllSheets)
- `#repeat-sheet` — repeat picker (slides over add view, z-index 400, shows overlay)
- `#recurring-sheet` — Edit Rule sheet; opens when tapping a rule row in Recurring tab (z-index 400, shows overlay). Contains: Since (read-only), Category (read-only), Note input, Amount display + numpad, Frequency picker, Save Changes button.
- `#cat-sheet` — ONLY used for "New Category" name-entry form (z-index 400, shows overlay); category grid is now inline in `#cat-inner`
- `#bottom-panel` — `position: sticky; bottom: calc(64px + env(safe-area-inset-bottom))` — sticks above the action bar when either panel is open; contains `#calc-inner` and `#cat-inner`; one shown at a time
- `#monthly-filter-sheet` — category picker for Monthly tab filter (z-index 300, standard sheet); opens via `openMonthlyCatPicker()`; lists only categories with actual transactions; "All categories" at top; sets `subSheet = 'monthly-filter'`
- `#photo-action` — Camera/Gallery action sheet (z-index 550, custom iOS-style)
- `.add-action-bar` — fixed bottom bar (z-index 50); Save + Delete buttons
- Overlay click: handles repeat/recurring/cat/monthly-filter sheets; inline panels do NOT use the overlay

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

- **Background:** `linear-gradient(150deg, #fdf4ff 0%, #eef2ff 45%, #f0fdfa 100%)` — applied to `#app` (NOT html/body)
- **html, body background:** `#ffffff` — matches nav bar so any PWA bleed-through is invisible
- **Header:** `rgba(255,255,255,0.88)` + `backdrop-filter: blur(20px)` (frosted glass)
- **Nav:** `rgba(255,255,255,0.98)` — fully opaque, NO frosted glass (removed Session 10 to prevent gradient bleed-through)
- **Header title (main views):** gradient text `linear-gradient(135deg, #7c3aed 0%, #ec4899 100%)`
- **Add-page header title:** plain `#111827` (no gradient) — use `.add-page-header-title` class
- **Primary UI accent:** `#7c3aed` (violet/purple) — FAB, nav active, save button, today dot, calc operators
- **Expense amounts in rows:** `#111827` black — red removed since all entries are expenses (no income)
- **`--expense` variable:** still `#ef4444` red — used for nav active, delete button, repeat badge, calendar dots; do NOT remove
- **Transaction cards:** `rgba(255,255,255,0.85)` glass, border-radius 16px, margin 0 12px 10px
- **Summary stat cards:** pink gradient (expenses), purple gradient (count)
- **Today highlight:** purple gradient (was dark navy)
- **Selected non-today:** `#ede9fe` bg, `#7c3aed` text (unchanged)

### Base variables (still in `:root`)
- `--bg: #f4f4f7` (fallback only — #app uses gradient directly)
- `--card: #ffffff`
- `--expense: #ef4444`
- `--today-bg: #1e293b` (overridden by theme)
- Font: Plus Jakarta Sans (Google Fonts CDN)
- Safe area: `env(safe-area-inset-top)` top, `env(safe-area-inset-bottom)` bottom

### App shell layout (Session 10)
- `#app` uses `position: fixed; top: 0; bottom: 0; left: 0; right: 0; overflow: hidden` — pins to exact viewport edges, no height calculations needed
- `#nav` uses `env(safe-area-inset-bottom, 0px)` DIRECTLY in `padding-bottom` and `height: calc(60px + env(...))` — NOT via `--safe-bottom` CSS variable (custom property + env() was unreliable on iOS)
- The bottom safe area gap (white space below nav icons) is the iOS home indicator area — correct behaviour, not a bug

### Form rows (Session 5 / Session 9)
- All `.form-row` elements: fixed `height: 44px`, `padding: 0 20px`, `box-sizing: border-box`
- `.form-input`: `font-size: 16px; font-weight: 600` — 16px is the iOS minimum to prevent auto-zoom on tap
- `.amount-input`: `font-size: 16px; font-weight: 600` — same as form-input (was 15px, caused zoom)
- `.desc-textarea`: `font-size: 16px`, `min-height: 90px` for memo typing
- Repeat button: icon-only (🔁), `width: 34px; height: 34px`, no label text

### Amount field (Session 9 change)
- `#f-amt` is now a `<span>`, NOT an `<input>` — this prevents iOS from focusing it and auto-zooming
- `onclick="openCalc()"` on the span opens the calculator
- `updateCalcDisplay()` uses `el.textContent` (not `el.value`) to update the display
- `syncFormState()` reads `am.textContent || am.value` to support both span and input
- Do NOT revert to `<input>` — iOS will zoom on focus even with `readonly` + `inputmode="none"`

### Date field (Session 9 change)
- Date row shows a `<span class="form-input">` with `fmtFormDate(fDate)` ("27 May 2026" format)
- A hidden `<input type="date" id="f-date">` sits absolutely positioned (opacity:0, 1×1px, pointer-events:none)
- Tapping the row calls `document.getElementById('f-date').focus()` to open the native iOS date picker
- `onchange` on the hidden input: `fDate=this.value; renderContent()` — re-renders the display span
- `syncFormState()` and `saveTx()` still read `da.value` from the hidden input — works as before
- Do NOT use native `<input type="date">` as the visible element — iOS centres its value and ignores text-align

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
