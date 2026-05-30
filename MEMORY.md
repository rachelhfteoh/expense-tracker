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

### Transaction layout — category left, note right (Session 5)
- Rachel wanted category (icon + label) on the left and note text on the right, not stacked.
- `.tx-left` is now `flex-direction: row`, `width: 120px` — icon (42px) + category label beside it.
- If no note, shows `—` rather than repeating the category name (which is already on the left).
- Both Transactions tab and Calendar day panel sort by `amount` descending (highest first).

### Save/Delete button bar — always both visible (Session 5)
- Rachel wanted both buttons always visible, not just Delete in edit mode.
- In add mode, Delete = discard (calls `closeAddView()`); in edit mode, Delete = confirm + remove.
- Fixed bottom bar at z-index 50 (below overlay at 200) — gets dimmed when sheets open.
- `height: 90px` spacer added at bottom of form to prevent action bar covering the calc panel.

### Category delete rules — tagged categories protected (Session 5)
- Rachel suggested: categories with tagged transactions cannot be deleted.
- `renderCatInner()` computes `usedKeys = new Set(data.transactions.map(t => t.category))` and only shows ✕ if `!usedKeys.has(c.key)`.
- Built-in cats go to `data.hiddenCats[]` (reversible); custom cats are fully removed from `data.customCats[]`.
- `getAllCats()` filters out `hiddenCats` so hidden built-in cats don't appear in the picker.

### Default category must not be 'other' (Session 5)
- `fCat = 'other'` was the hardcoded default — but 'other' is a hidden fallback, not a real category.
- Changed all three init points (`let fCat`, `openAdd()`, `openAddForDay()`) to `'groceries'`.

### Week strip — idempotent swipe init (Session 5)
- `initTxWeekSwipe()` guards with `strip._swipeInit = true` to prevent duplicate event listeners on each re-render.
- `renderTxWeekStrip()` is called by `renderContent()` after transactions HTML is set — the `#tx-week-strip` div must exist in the HTML at that point.
- Week strip uses Sun-first (S M T W T F S) to match the Calendar tab grid.

### Week strip class missing — was rendering vertically (Session 6)
- `#tx-week-strip` div was missing `class="week-strip"` so the grid CSS never applied — children stacked vertically.
- Fix: `<div id="tx-week-strip" class="week-strip">` — both id AND class required.
- Lesson: when a layout is wrong (vertical vs horizontal), first check if the CSS class is actually applied to the element.

### Transactions tab now filters by selected day, not full month (Session 6)
- Old: `buildTransactions()` showed all transactions for `txForMonth()`, grouped by date.
- New: shows only `txForDay(txSelDay)` — one day at a time, selected via the week strip.
- `selectTxDay()` now just updates state and calls `renderContent()` — no scroll logic needed.
- `changeTxWeek()` must call `renderContent()` not just `renderTxWeekStrip()` — otherwise the transaction list doesn't update when week changes.
- FAB date defaults to `txSelDay` not `todayStr()` when opening from transactions view — critical, otherwise expense saves to wrong day.

### Category detail view — new full-page view (Session 6)
- `view = 'cat-detail'` follows same pattern as `view = 'add'`: nav/FAB hidden, back button in header.
- `catDetailKey` stores the category being viewed; `stYear/stMon` provides month context.
- `openEdit()` from cat-detail sets `addViewPrev = 'cat-detail'` — after save, `render()` returns to cat-detail correctly because `renderContent()` handles `view === 'cat-detail'`.
- `catDetailRowHtml()` is a separate compact renderer (no icon/label) — do NOT reuse `txRowHtml()` for cat-detail rows.

### Amount colour changed to black — `--expense` variable kept (Session 6)
- `.tx-amount` and `.day-total` changed to `#111827` (black) — red removed since all entries are expenses.
- `var(--expense)` #ef4444 kept in the CSS variable — still used by nav active, delete button, repeat badge, calendar dots. Do NOT remove or rename it.
- `.cd-row .tx-amount` scoped override for cat-detail font-size — both are now 13px.

### Number formatting with commas (Session 7)
- `rm(n)` changed from `Number(n).toFixed(2)` to `Number(n).toLocaleString('en-MY', { minimumFractionDigits: 2, maximumFractionDigits: 2 })`.
- This applies everywhere `rm()` is called — rows, totals, calendar, stats, cat-detail — no other changes needed.

### Donut chart labels — SVG font size scaling trap (Session 7)
- When SVG display size < viewBox size, all coordinates and font sizes scale down proportionally.
- viewBox 400×300 displayed at 300×225 = 0.75× scale. Font-size 10px SVG → 7.5px on screen (unreadable).
- Fix: use `overflow="visible"` on the SVG so labels aren't clipped, and set font-size to compensate (e.g. 16px SVG → 12px rendered at 0.75× scale).
- Labels use `<tspan>` with `dy` offsets for two-line layout (name on top, % below in category colour).

### Blank category default — validation pattern (Session 7)
- `fCat = ''` in all three init points: `let fCat`, `openAdd()`, `openAddForDay()`.
- Category row shows "Pick a category" in grey when `fCat` is empty (`selCat = fCat ? getCat(fCat) : null`).
- `saveTx()` validates: `if (!fCat) { toast('Pick a category'); return; }` — same pattern as amount/date validation.
- `deleteCatInline()` resets `fCat = ''` (not `'groceries'`) if the deleted cat was selected.
- Edit mode is unaffected — `fCat = t.category` is always a real key from stored data.

### Recurring overlay freeze bug (Session 7)
- `pickRepeat(key)` calls `closeRepeatSheet()` then `renderContent()`.
- `closeRepeatSheet()` was only removing the sheet's `.show` class — it did NOT hide the overlay.
- Overlay stayed visible and blocked all touches, making the screen appear frozen.
- Fix: add `if (view === 'add') document.getElementById('overlay').classList.remove('show');` inside `closeRepeatSheet()`.
- Root cause: `overlayClick()` handled overlay removal for overlay taps, but `pickRepeat()` bypassed it.

### Bottom panel sticky positioning (Session 7)
- `#bottom-panel` changed to `position: sticky; bottom: calc(64px + env(safe-area-inset-bottom)); z-index: 10`.
- This makes calc/cat panel stick to the bottom of the visible `#content` viewport, above the fixed action bar.
- No `scrollIntoView` needed — sticky handles it automatically.
- The 90px spacer div below `#bottom-panel` remains to prevent the action bar covering content when panel is closed.

### Calculator and category panel no longer auto-open (Session 7)
- Removed `openCalc()` from `openAdd()`, `openAddForDay()`, and `openEdit()`.
- Both panels now open only on user tap (Amount → calc, Category → cat grid).
- `pickCatInline()` sets `activePanel = null` after pick (was `'calc'`) — avoids calc snapping back after category selection.

### Month navigation and week strip sync bug (Session 8)
- `prevTx()`/`nextTx()` previously only changed `txMon/txYear` — `txSelDay` and `txWeekOffset` stayed at today's values.
- Result: navigating to April showed April in the header but the week strip and transaction list still showed May's current week.
- Fix: `syncTxToMonth()` sets `txSelDay = lastDayOfMonth(txYear, txMon)` (capped at today) then computes `txWeekOffset = weekOffsetForDate(txSelDay)`.
- `weekOffsetForDate(ds)` computes integer weeks between this week's Sunday and ds's Sunday — always negative for past weeks.

### Future month blocking — all three tabs (Session 8)
- `nextTx()`, `nextCal()`, `nextSt()` all return early if already at current year/month.
- The `>` button is dimmed to `opacity: 0.3` and `disabled` when at current month — set after rendering the nav label.
- Query: `document.querySelectorAll('#tx-nav .mnav-btn')[1]` — index 1 = right button.

### Inline + button in Transactions day header (Session 8)
- FAB hidden in `view === 'transactions'` and `view === 'stats'` — `isDetail` already hides for cat-detail.
- Day header always renders first (even when empty), then either the empty state or the tx-card below.
- The empty state `<div class="empty">` sits inside `day-group` below the header — not a top-level sibling.
- `+ button` calls `openAdd()` (not `openAddForDay`) — `openAdd()` already defaults `fDate` to `txSelDay`.

### Stats layout — fixed-width columns for alignment (Session 8)
- Removed progress bars from stats category rows — % alone is sufficient, bar was redundant.
- Removed coloured dots — emoji already identifies the category.
- `stats-pct`: `width: 44px; text-align: right` — wide enough for "100%".
- `stats-amt`: `width: 110px; text-align: right` — wide enough for "RM 999,999.00" at 13px bold.
- Without fixed widths, variable amount lengths push the % column left/right per row — looks misaligned.

### Transaction row font size and category colour (Session 8)
- All tx row elements unified to 12px: `.tx-note`, `.tx-cat`, `.tx-amount`, `.cd-row .tx-amount`.
- `.tx-cat` colour changed from `var(--text-3)` to `#6b7280` — more readable at 12px without overpowering the note.
- `.tx-left` width increased from 95px to 120px to fit 13-character category names without truncation.

### Deployment to GitHub Pages (Session 9)
- Single `index.html` project deploys directly to GitHub Pages with no build step.
- Steps: create repo on GitHub → `git remote add origin <url>` → `git push -u origin main` → Settings → Pages → Branch: main / (root) → Save.
- Takes ~60 seconds to go live at `https://<username>.github.io/<repo>/`.
- To update: just `git push` — GitHub Pages auto-redeploys on every push to main.
- PWA icon: add `apple-touch-icon.png` (180×180) to repo root + `<link rel="apple-touch-icon" href="apple-touch-icon.png">` in `<head>`.
- iPhone users must delete the old home screen icon and re-add after icon changes — Safari caches the old icon.

### iOS auto-zoom on input tap (Session 9)
- iOS Safari auto-zooms any `<input>` or `<textarea>` with `font-size < 16px` when tapped.
- Fix: set ALL input/textarea font-sizes to minimum 16px. Even `readonly` inputs with `inputmode="none"` trigger zoom.
- The amount field (`#f-amt`) was changed from `<input readonly>` to `<span onclick="openCalc()">` — this completely prevents focus and zoom since spans are not focusable.
- `updateCalcDisplay()` uses `el.textContent` not `el.value` now that `#f-amt` is a span.
- Calculator buttons need `touch-action: manipulation` to prevent double-tap zoom.

### iOS date input alignment (Session 9)
- `<input type="date">` on iOS Safari always centres its value text, regardless of `text-align: left` or `-webkit-appearance: none`. CSS cannot override this.
- Fix: replace the visible date input with a `<span>` showing `fmtFormDate(fDate)`, and a hidden `<input type="date" id="f-date">` (opacity:0, 1×1px, pointer-events:none).
- Tapping the row calls `.focus()` on the hidden input to open the native iOS date picker.
- `syncFormState()` and `saveTx()` still read `da.value` from the hidden input — no other code changes needed.
- `onchange` fires when user picks a date, updates `fDate`, calls `renderContent()` to refresh the display span.

### PWA icon generation (Session 9)
- No image editing tools available — icons generated with raw Python using `zlib` + `struct` modules.
- PNG format: signature + IHDR chunk (width, height, 8-bit RGB) + IDAT chunk (zlib-compressed pixel rows) + IEND chunk.
- Each row: filter byte (0) + RGB pixels. Chunks: 4-byte length + tag + data + CRC32.
- App icon design: dark `#111111` background, gradient circle (purple #7c3aed → pink #ec4899), white "E", two gradient dots below. Matches dark iPhone theme.

### PWA bottom gap — iOS home indicator safe area (Session 10)
- The white space below the nav icons is the iOS home indicator safe area (~34px) — correct PWA behaviour, not a bug.
- `#app` changed to `position: fixed; top: 0; bottom: 0; left: 0; right: 0; overflow: hidden` — the most reliable way to pin a PWA shell to exact viewport edges.
- Gradient moved from `html, body` to `#app` — CSS spec paints `html`/`body` backgrounds to the full viewport canvas regardless of their computed height, causing bleed-through. Gradient on `#app` is bounded by the element.
- `html, body { background: #ffffff }` — matches nav bar so any residual bleed-through is invisible.
- `env(safe-area-inset-bottom)` must be used DIRECTLY in nav height/padding — using it via a CSS custom property (`--safe-bottom: env(...)`) was unreliable on this device.

### Swipe-to-delete on transaction rows (Session 10)
- Each `.tx-row` is wrapped in `.tx-row-wrap` (overflow:hidden) with `.tx-swipe-del` absolutely positioned at the right edge.
- `.tx-row` must have `background: var(--card)` — it's the "cover" that hides the red delete button when not swiped.
- `.tx-row:last-child { border-bottom: none }` rule REMOVED — since `.tx-row` is always the last child of its `.tx-row-wrap`, this rule matched every row. Use `.tx-row-wrap:last-child .tx-row` instead.
- `touchmove` uses `{ passive: false }` and calls `e.preventDefault()` when swipe is primarily horizontal — prevents vertical scroll hijacking.
- `_openWrap` module variable tracks the currently open row; `closeOpenSwipeRow()` resets it. Always call `closeOpenSwipeRow()` in `switchView()`.
- `touch-action: pan-y` on `.tx-row-wrap` — required so iOS Safari passes horizontal swipe events to JS even when the content area is scrollable (e.g. Calendar tab).
- `initSwipeRows()` must be called after BOTH `view === 'transactions'` AND `view === 'calendar'` renders — both views use `txRowHtml()`.

### Decimal-first amount input (Session 11)
- Amount input uses cash register / POS style: digits flow right-to-left, always 2 decimal places.
- State: `calcCents` (integer cents), `calcLeftCents` (left operand), `calcOp` (pending operator or null).
- `openCalc()` seeds `calcCents = Math.round(parseFloat(fAmount) * 100)` so edit mode loads correctly.
- `updateCalcDisplay()` formats via `centsToStr()` and syncs `fAmount = (calcCents/100).toFixed(2)`.
- The `.` button was replaced with `00` (double zero) — more useful for round amounts in cash-register style.
- `syncFormState()` and `saveTx()` strip commas before parsing: `.replace(/,/g, '')` — the display shows "1,234.56" but fAmount stores "1234.56".
- `buildAddPage()` formats the initial amount display with `toLocaleString` for consistency.

### Recurring tab and rule management (Session 12)
- Added 4th nav tab `view = 'recurring'` — `buildRecurring()` / `deleteRuleFromView(id)`.
- `renderHeader()` has a new `else if (view === 'recurring')` branch before the `else` (Stats) branch — title only, no month nav.
- **Rule sync on edit:** `saveTx()` edit path now also updates `rule.note`, `rule.amount`, `rule.category` when `tx.recurringId` exists. Without this, Recurring tab shows stale data after editing a transaction.
- **Smart 🔁 badge:** `txRowHtml()` and `catDetailRowHtml()` check `data.recurring.some(r => r.id === t.recurringId)` — badge only shows while the rule is alive. Do NOT simplify back to `t.recurringId ? ...` or the badge will persist after rule deletion.
- **`recurringId` is NOT nulled on rule delete** — it's kept on transactions so the 🔁 indicator still shows in the Edit Expense header (`fIsRecurring = !!t.recurringId` in `openEdit()`). Nulling it would break this feature.
- **Delete rule UX:** `deleteRuleFromView()` uses `confirm()` with message "Delete all future recurring transactions?". Deletes future transactions (`date > today`) + removes rule. Today and past entries always preserved.
- **"Future" = strictly after today** — use `date > today` (not `>=`). Today's transaction is "current" and kept.

### Recurring tab improvements (Session 13)
- **Edit Rule sheet:** `#recurring-sheet` repurposed as the rule edit sheet (was dead code). `openRuleEdit(id)` populates state and opens it. `renderRuleEditSheet()` builds the form. `saveRuleEdit()` validates, updates rule, syncs today+future transactions, regenerates if frequency changed.
- **Rule edit amount uses cash register numpad** — same `ruleEditCents` / `ruleCalcInput()` / `ruleCalcBack()` pattern as main calc. DO NOT use a plain `<input>` for rule amount — user expects decimal-first format.
- **Delete recurring transaction** — `swipeDeleteTx()` and `deleteTx()` now: (1) always remove the specific transaction first by ID, then (2) if `tx.recurringId` exists, remove rule + instances `date > today`. The two steps are independent — step 1 doesn't depend on date filters.
- **Bug fix: filter logic for deleting recurring transactions** — original attempt used `date >= today` filter to remove the transaction, which failed for past-dated entries. Fix: always remove the specific transaction by ID first (`filter t.id !== fEditId`), then separately clean up future instances (`date > today`).
- **🔁 button visibility in Edit Expense** — shown when `fMode === 'add' || !fIsRecurring`. Hidden for existing recurring transactions (their rule is managed from the Recurring tab). Shown for non-recurring transactions so you can retroactively add a repeat.
- **Adding repeat while editing a non-recurring transaction** — `saveTx()` edit path now checks `fRepeat !== 'none' && !tx.recurringId` → creates a new rule and sets `tx.recurringId`. Previously only synced existing rules.
- **deleteRuleFromView() updated** — now deletes instances `>= today` (today inclusive, not just future). Confirm message updated to "Delete this and all future recurring transactions?".
- **Rule rows in Recurring tab are tappable** — `onclick="openRuleEdit(...)"` on `.rule-row`. Delete button uses `event.stopPropagation()` to prevent triggering the row tap.

### Monthly tab and export/import (Session 14)
- **Monthly tab** is the 5th nav tab (`view = 'monthly'`). `renderHeader()` needs an `else if (view === 'monthly')` branch BEFORE the `else` (Stats) branch — same pattern as recurring.
- **SVG line chart x-axis labels:** show ALL labels when n ≤ 6 months; show 5 evenly spaced when n > 6. Original code only showed first + last for n < 4, causing April to be hidden with 3 months of data.
- **Export/Import lives in Monthly tab** behind a settings gear ⚙ icon that opens `#data-action` action sheet — same iOS-style sheet as `#photo-action`. Reuses `.photo-action-*` CSS classes.
- **PWA standalone storage is isolated from Safari browser storage** — data entered via the home screen icon is NOT visible when opening the same URL in Safari. Always use the home screen icon. Export must also be done from the home screen icon.
- **Deleting the home screen PWA icon on iOS also deletes its standalone localStorage** — always export before deleting the icon, or data is permanently lost.
- **Recurring frequency simplified to Monthly only** — quarterly/bi-annual/annual removed. Monthly Commitment = straight sum of all rule amounts (no division). Frequency key `'monthly'` is the only valid value now.
- **Recurring sub-line format** — changed from `freq · amount · since date` to `category · freq · amount`. "Since" date removed — it's visible in the Edit Rule sheet.
- **Monthly Commitment in Recurring summary bar** — straight sum of all `data.recurring` amounts. No normalisation needed since all rules are monthly frequency only.

### Always commit and push before asking Rachel to test (Session 11)
- Rachel tests exclusively on GitHub Pages (https://rachelhfteoh.github.io/expense-tracker/), not local files.
- Every fix must be committed and pushed BEFORE asking her to test — otherwise she sees the old cached version.
- This cost a full session of confusion — she was testing on stale cache the entire time thinking fixes weren't working.

---

## Things to Watch Out For

- **localStorage is browser-specific.** Always test in Safari — data saved in Safari won't appear in Chrome.
- **syncFormState() before sub-sheets:** Must sync form field values into state variables BEFORE opening cat-sheet or repeat-sheet, or those fields reset to stale state on re-render.
- **Sun-Sat calendar column colours:** These rely on CSS `nth-child` selectors — `7n+1` = Sun (col 1), `7n` = Sat (col 7). Don't change the grid column order.
- **End of month edge case:** Day 0 of month M+2 = last day of month M+1. Use `new Date(y, m+2, 0)` pattern.
- **Calc keyboard state:** `calcCents` is the source of truth. `openCalc()` seeds it from `fAmount`. `updateCalcDisplay()` always syncs both `#f-amt` display and `fAmount` string. After `renderContent()` (e.g. category change), amount field re-renders from `fAmount` — this is correct.
- **CSS theme override order matters:** The colourful theme block must come AFTER the base styles in `<style>` or the cascade won't work. Never move base `:root` variables below the overrides.
- **Add-page title uses plain font, not gradient:** `.add-page-header-title` is plain `#111827`. The gradient text styling is on `.header-title` which is only used in main views.
- **buildAddPage() returns HTML string** — do not try to call `document.getElementById('add-body')` in add-view flow; that element is in the unused `#add-sheet`. The form is rendered into `#content`.
- **Panel toggle is DOM-only, not re-render** — `openCalc()` and `openCatPanel()` directly set `element.style.display`. Never call `renderContent()` just to switch panels — it causes unnecessary DOM rebuilds and potential focus loss on text fields.
- **renderCatInner() must be called to update selection** — the cat grid in `#cat-inner` is not rebuilt by `initPanel()` automatically; it's built fresh by `renderCatInner()` each time the cat panel opens. After `pickCatInline()` calls `renderContent()`, `initPanel()` handles it.
- **Amount 0.00 default** — `fAmount = '0.00'` when adding. `calcInput()` clears it on first digit. `calcBack()` clears in one tap. `calcEqual()` always formats to 2dp. Edit view uses `Number(t.amount).toFixed(2)` so stored amounts also display as 2dp.
- **`#f-amt` is a span, not an input** — never revert to `<input>` or iOS will zoom on focus. All reads use `el.textContent`; `syncFormState` handles both span and input via `am.textContent || am.value`.
- **Date row has hidden input** — `#f-date` is `opacity:0; width:1px; height:1px; pointer-events:none`. Tapping the row calls `.focus()` on it. Do not remove or make it `display:none` — it must remain in the DOM and focusable for the iOS date picker to work.
- **All form inputs must be ≥ 16px** — iOS auto-zooms anything smaller. Never reduce `.form-input`, `.amount-input`, or `.desc-textarea` below 16px.
- **FAB date must follow txSelDay** — `openAdd()` uses `txSelDay` when `view === 'transactions'`. If this ever reverts to `todayStr()`, expenses added from a past day will silently save to today and not appear.
- **Theme override `.day-header` padding** — there is a theme CSS override for `.day-header` padding (near bottom of `<style>`). If day headers look too spaced, check this override — the base style alone is not enough.
- **`#app` is `position: fixed`** — do not change this to `height: 100%` or `height: 100dvh`. Fixed positioning is the only reliable way to pin the PWA shell on iOS without viewport height ambiguity.
- **Nav bar background is opaque** — `rgba(255,255,255,0.98)`, no backdrop-filter. Do not restore frosted glass to the nav; it caused gradient bleed-through in the icon area.
- **`env()` in nav must be direct** — use `env(safe-area-inset-bottom, 0px)` inline in `#nav` height and padding-bottom. Do not use `var(--safe-bottom)` — it was silently resolving to 0px on this device.
- **Swipe rows: `.tx-row-wrap:last-child .tx-row`** — use this selector for removing the last border, NOT `.tx-row:last-child` which matches every row in the new wrapper structure.
- **🔁 badge check must use `data.recurring.some()`** — never simplify to `t.recurringId ? ...`. The rule may have been deleted while the transaction still has a recurringId.
- **`fIsRecurring` must be reset in openAdd() and openAddForDay()** — if not reset, stale `true` from a prior edit bleeds into a new-expense form and shows 🔁 in the header incorrectly.
- **Delete recurring transaction: remove by ID first, then clean up** — never rely solely on a date filter to remove the target transaction. Always `filter(t => t.id !== id)` first, then separately `filter` future instances. If you only use the date filter, past-dated recurring transactions won't be removed.
- **Edit Rule sheet amount is `ruleEditCents` (integer cents)** — `saveRuleEdit()` reads `ruleEditCents / 100`. Never revert to a string `ruleEditAmt` — the numpad doesn't write to a string anymore.
- **`#recurring-sheet` is now the Edit Rule sheet** — it opens from the Recurring tab (not from Transactions header). `openRecurringSheet()` and `renderRecurringSheet()` are replaced by `openRuleEdit()` and `renderRuleEditSheet()`. Do not restore the old functions.
- **PWA standalone storage ≠ Safari browser storage** — iOS gives PWA home screen apps their own isolated localStorage. Opening the GitHub Pages URL in Safari always shows empty data. Remind Rachel to always use the home screen icon.
- **Deleting the home screen icon wipes PWA data on iOS** — always export backup first. Do not ask Rachel to delete and re-add the icon without exporting first.
