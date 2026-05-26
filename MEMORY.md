# MEMORY.md — Expense Tracker

Lessons learned, key decisions, and things to remember for future sessions.

---

## Design Decisions

### Expenses only — no income tracking
- Rachel's income is a fixed salary; no need to track it in the app.
- All transaction amounts are positive; no negative values or income/expense toggle.

### MYR only
- No multi-currency support. All amounts displayed as "RM X.XX".

### Sun–Sat calendar (same as habit tracker)
- Calendar week starts Sunday. Rachel confirmed this matches her habit tracker.
- Calendar grid: col 1 = Sun (red text), col 7 = Sat (blue text).
- `new Date(y, m, 1).getDay()` gives 0=Sun which is correct for the Sun-first layout.

### No account/wallet tracking
- Rachel does not track which card/bank account was used. Account field deliberately omitted.

### Recurring — edit this instance only (Option C)
- Editing a recurring transaction changes only that one instance.
- To change the recurring rule itself, use "🔁 Recurring" in the Transactions header.
- Deleting a rule stops future generation; existing instances remain.

### Colourful theme matches Habit Tracker (Session 2)
- Rachel wanted the expense tracker to look like her habit tracker — same purple/lavender gradient, same purple accent colour.
- Expense amounts stay red (semantically: red = money spent). UI chrome (FAB, nav, buttons, today dot) uses purple `#7c3aed`.
- Theme overrides added as a CSS block near the bottom of `<style>` — don't move them above the base styles or cascade breaks.

### Calculator keyboard instead of native keyboard (Session 2)
- Rachel saw a reference app with an in-app calculator that pops up for the amount field.
- Custom `#calc-keyboard` (fixed, z-index 500) slides up from the bottom like a native keyboard.
- Supports `+`, `−`, `×`, `÷`, preview of running total, `=` to evaluate, OK to confirm.
- Amount field uses `type="text" inputmode="none" readonly` — this suppresses the native keyboard on iOS.

---

## Technical Decisions

### "End of Month" frequency — snap startDate
- When saving with "End of Month", the transaction date and rule startDate are snapped to the last day of the selected month (not the day the user chose).
- `nextOccurrence` for `endofmonth` returns the last day of the month after the input date.

### addMonths helper — clamps to valid day
- `addMonths("2026-01-31", 1)` → "2026-02-28" (not March 3).
- Always clamp: `Math.min(d, daysInMonth(ny, nm))`.

### Recurring generation safety guard
- `generateRecurring()` has a 1000-iteration cap per rule to prevent runaway loops for high-frequency rules on large gaps.

### subSheet state tracks sheet stacking
- `subSheet` variable (`'repeat' | 'recurring' | null`) tracks which secondary sheet is open above the add sheet.
- Overlay click is smart: closes only the topmost sheet.

### renderContent() vs render()
- `render()` rebuilds everything (header + content + nav).
- `renderContent()` rebuilds only the content area — used for calendar day selection to avoid re-running initSwipe on the header.

### evalAmount uses Function() with whitelist — safe for local app
- `evalAmount(raw)` whitelists input to `/^[\d\.\+\-\*\/\(\)]+$/` before passing to `Function()`.
- This is safe because the app is fully local (no server, no other users). Never use this pattern in a server-side context.

### Amount input must be type="text", not type="number"
- `type="number"` prevents storing expression strings like `3.75+4.95` — the browser sanitises the value.
- Use `type="text" inputmode="none"` to support both plain numbers and expressions.

### overlayClick must check calc keyboard first
- `#calc-keyboard` sits at z-index 500. When it's open, tapping the overlay behind the add sheet should close the calc only — not close the entire sheet stack.
- Always add `if (calc.show) { closeCalc(); return; }` as the first check in `overlayClick()`.

---

## Things to Watch Out For

- **localStorage is browser-specific.** Always test in Safari — data saved in Safari won't appear in Chrome.
- **Category grid re-render on pickCat:** Must sync form field values (date, amount, note) into state variables BEFORE calling `renderAddSheet()`, or those fields reset to stale state.
- **Sun-Sat calendar column colours:** These rely on CSS `nth-child` selectors — `7n+1` = Sun (col 1), `7n` = Sat (col 7). Don't change the grid column order.
- **End of month edge case:** Day 0 of month M+2 = last day of month M+1. Use `new Date(y, m+2, 0)` pattern.
- **Repeat sheet stacks above add sheet:** `#repeat-sheet` has `z-index: 400`, `#add-sheet` has `z-index: 300`. Overlay stays open when repeat sheet closes so add sheet remains visible.
- **Calc keyboard state:** `calcExpr` and `fAmount` must stay in sync. `openCalc()` seeds `calcExpr` from `fAmount`. `calcInput/calcBack/calcEqual` update both. If `renderAddSheet()` is called while calc is open (e.g. category change), the amount field re-renders from `fAmount` — this is correct.
- **CSS theme override order matters:** The colourful theme block must come AFTER the base styles in `<style>` or the cascade won't work. Never move base `:root` variables below the overrides.
