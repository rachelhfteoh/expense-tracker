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
- Calculator is inline in the form (not fixed-position), shown/hidden via display:none/block.
- Supports `+`, `−`, `×`, `÷`, preview of running total, `=` to evaluate, OK to confirm.
- Amount field uses `type="text" inputmode="none" readonly` — this suppresses the native keyboard on iOS.

### Inline bottom panel — calc and category share same space (Session 4)
- Rachel wanted the category grid to appear at the bottom replacing the calculator, same as reference app.
- `#bottom-panel` contains two children: `#calc-inner` and `#cat-inner` — only one shown at a time.
- Tapping Amount → `openCalc()` → shows `#calc-inner`. Tapping Category → `openCatPanel()` → shows `#cat-inner`.
- Toggle is direct DOM show/hide — NO re-render. This avoids focus loss when Note/Desc field is active.
- `initPanel()` must be called after every `renderContent()` DOM rebuild to restore the correct panel.
- `activePanel` state (`'calc' | 'cat' | null`) tracks which panel should be visible; always reset to `null` before entering add view, then `openCalc()` sets it.

### Compact form + category panel (Sessions 3–4)
- Rachel wanted the add form to be compact — no visible category grid on screen at first.
- Category tapping now opens the inline `#cat-inner` panel (not a separate sheet).
- `#cat-sheet` is now ONLY used for the "New Category" name-entry form.
- Custom categories are stored in `data.customCats[]` with `key: 'custom_<uid>'`, emoji auto-assigned '📦', color auto-from palette.
- "Other" is hidden from the category picker but kept as a code fallback in `getCat()`.
- "Add" tile at the end of the inline grid opens `#cat-sheet` in name-only mode.

### Full-page add view instead of bottom sheet (Session 3)
- Rachel wanted tapping + to open a full page, not slide up a sheet from the bottom.
- Implemented as `view = 'add'` — the same render loop handles it like any other view.
- Header switches to: back button (← Trans./Calendar/Stats) | title | Save button.
- Nav bar and FAB are hidden when `view === 'add'`.
- Calculator auto-opens (`openCalc()`) immediately when entering the add view.
- Date, Note, Description inputs have `onfocus="closeCalc()"` so system keyboard can take over when those fields are tapped.

### Description + photo (Session 3)
- Rachel wanted a long-text description field below Note, and a camera button to attach a photo.
- Description is a `<textarea>` with `resize: none` and auto-grows with content.
- Camera button opens a custom iOS-style action sheet (`#photo-action`) with Camera / Gallery options.
- Two hidden `<input type="file">` elements: one with `capture="environment"` (camera), one without (gallery).
- Photos are compressed to max 800px JPEG 70% via Canvas before storing as base64 in localStorage.

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
- `subSheet` variable (`'repeat' | 'recurring' | 'cat' | null`) tracks which secondary sheet is open.
- Overlay click is smart: closes only the topmost sheet.
- In `view === 'add'` mode, the overlay is shown by sub-sheets (cat, repeat) and hidden when they close.

### renderContent() vs render()
- `render()` rebuilds everything (header + content + nav).
- `renderContent()` rebuilds only the content area — used for calendar day selection, category pick, repeat pick.
- After picking a category or repeat frequency in the add view, call `renderContent()` not `render()` (avoids re-running initSwipe).

### syncFormState() must be called before opening any sub-sheet
- `syncFormState()` reads the live DOM values of date, amount, note, description into JS state.
- Must be called in `openCatSheet()` and `openRepeatSheet()` before the sub-sheet opens.
- If skipped, the sub-sheet close + re-render will restore stale state, losing whatever the user typed.

### evalAmount uses Function() with whitelist — safe for local app
- `evalAmount(raw)` whitelists input to `/^[\d\.\+\-\*\/\(\)]+$/` before passing to `Function()`.
- This is safe because the app is fully local (no server, no other users). Never use this pattern in a server-side context.

### Amount input must be type="text", not type="number"
- `type="number"` prevents storing expression strings like `3.75+4.95` — the browser sanitises the value.
- Use `type="text" inputmode="none"` to support both plain numbers and expressions.

### overlayClick — inline panels do NOT use the overlay (Session 4)
- The calc and category inline panels never show the overlay — `overlayClick()` no longer checks for them.
- Overlay is only shown for `#cat-sheet` (name form), `#repeat-sheet`, `#recurring-sheet`.
- In add-page view, overlay shown by sub-sheets only; must hide when they close.
- `#add-sheet` remains in HTML but is never shown — prevents null-ref errors in `closeAllSheets()`.

### initPanel() critical after every renderContent() in add view (Session 4)
- `renderContent()` rebuilds `#content` HTML, creating fresh `#calc-inner` and `#cat-inner` elements with `display:none`.
- `initPanel()` must be called immediately after to re-apply `activePanel` state.
- Forgetting this = panel disappears after any re-render (e.g. picking a category or adding a photo).

### activePanel must be reset before entering add view (Session 4)
- Set `activePanel = null` at the start of `openAdd()`, `openAddForDay()`, `openEdit()` before calling `render()`.
- Then `openCalc()` sets it to `'calc'`. If not reset, stale panel state from a prior session bleeds into the new DOM.

### Photo storage — base64 in localStorage
- Storing photos as base64 JPEG in localStorage is acceptable for a local app.
- Always compress first (max 800px, 70% quality) to avoid hitting the ~5MB localStorage quota.
- If localStorage quota is exceeded, the `save()` function will silently fail (no try/catch currently — worth adding in future).

---

## Things to Watch Out For

- **localStorage is browser-specific.** Always test in Safari — data saved in Safari won't appear in Chrome.
- **syncFormState() before sub-sheets:** Must sync form field values into state variables BEFORE opening cat-sheet or repeat-sheet, or those fields reset to stale state on re-render.
- **Sun-Sat calendar column colours:** These rely on CSS `nth-child` selectors — `7n+1` = Sun (col 1), `7n` = Sat (col 7). Don't change the grid column order.
- **End of month edge case:** Day 0 of month M+2 = last day of month M+1. Use `new Date(y, m+2, 0)` pattern.
- **Calc keyboard state:** `calcExpr` and `fAmount` must stay in sync. `openCalc()` seeds `calcExpr` from `fAmount`. `calcInput/calcBack/calcEqual` update both. After `renderContent()` (e.g. category change), amount field re-renders from `fAmount` — this is correct.
- **CSS theme override order matters:** The colourful theme block must come AFTER the base styles in `<style>` or the cascade won't work. Never move base `:root` variables below the overrides.
- **Add-page title uses plain font, not gradient:** `.add-page-header-title` is plain `#111827`. The gradient text styling is on `.header-title` which is only used in main views.
- **buildAddPage() returns HTML string** — do not try to call `document.getElementById('add-body')` in add-view flow; that element is in the unused `#add-sheet`. The form is rendered into `#content`.
- **Panel toggle is DOM-only, not re-render** — `openCalc()` and `openCatPanel()` directly set `element.style.display`. Never call `renderContent()` just to switch panels — it causes unnecessary DOM rebuilds and potential focus loss on text fields.
- **renderCatInner() must be called to update selection** — the cat grid in `#cat-inner` is not rebuilt by `initPanel()` automatically; it's built fresh by `renderCatInner()` each time the cat panel opens. After `pickCatInline()` calls `renderContent()`, `initPanel()` handles it.
- **Amount 0.00 default** — `fAmount = '0.00'` when adding. `calcInput()` clears it on first digit. `calcBack()` clears in one tap. `calcEqual()` always formats to 2dp. Edit view uses `Number(t.amount).toFixed(2)` so stored amounts also display as 2dp.
