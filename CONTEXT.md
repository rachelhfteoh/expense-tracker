# CONTEXT.md — Expense Tracker

## Current Status
App is functional and polished. Number formatting, donut chart labels, blank category default, recurring bug fix, and UI cleanup all complete. Tested in Safari.

## In Progress
Nothing in progress.

## Up Next
1. Deploy to GitHub Pages (PWA apple-touch-icon needed)
2. End-to-end test — recurring, calendar, stats drill-down, photo, week strip navigation

## Backlog
- Search / filter transactions
- Export data (JSON backup)
- Budget targets per category

## Completed — Archive
✅ Requirements gathered — MYR only, expenses only, Sun-Sat calendar, 14 repeat frequencies
✅ index.html built — Transactions, Calendar, Stats, add/edit/delete, recurring generation on load
✅ CLAUDE.md, CONTEXT.md, MEMORY.md initialised
✅ Initial git commit made
✅ Colourful theme — gradient background, purple accent, glass cards, pastel stat cards
✅ Design polish — SVG icons, larger touch targets, nav indicator line, transitions
✅ Calculator keyboard — custom in-app calc; supports +−×÷, preview, OK to confirm
✅ Compact add form — Date, Amount, Category row, Note, Description + camera icon
✅ Category sheet — bottom sheet with custom categories (emoji auto-assigned, color from palette)
✅ Description + photo — textarea, camera button, photo compressed to base64
✅ Full-page add view — view='add'; header back + Save; nav hides
✅ Inline bottom panel — calc and category grid share same panel space; toggle by tapping Amount or Category
✅ Amount defaults to 0.00 — clears on first digit; = and OK always format to 2dp
✅ "Other" hidden from picker — still used as code fallback; "Add" tile opens name-entry sheet
✅ Form row heights fixed — all rows 44px fixed height; amount font normalised to 15px/700
✅ Description textarea enlarged — min-height 90px for memo typing
✅ Repeat button icon-only — label removed, 🔁 icon only, title attribute for tooltip
✅ Default category changed to blank — was Groceries; now blank with "Pick a category" placeholder; Save shows toast if not picked
✅ Save/Delete bottom action bar — fixed bar at bottom; Save left, Delete right; both always visible
✅ Transaction layout restructured — category icon+name on left, note in middle, amount on right
✅ Sort by amount descending — highest spend first within each day group (Transactions + Calendar)
✅ Category grid 4 columns — was 3 columns; tiles smaller and phone-friendly
✅ Category delete rules — only deletable if no transactions tagged; built-in + custom both respect this
✅ Week strip fixed — was vertical due to missing class="week-strip"; now horizontal S M T W T F S
✅ Day filtering on Transactions tab — tapping a strip day shows only that day's transactions
✅ FAB date defaults to selected day — adding expense from transactions view uses txSelDay not today
✅ Category detail view — tapping a Stats category opens full-page breakdown (view='cat-detail')
✅ Compact row styling — icon 32px, font 13px across category/note/amount, day totals black not red
✅ Number formatting with commas — rm() uses toLocaleString; RM 7,000.00 not RM 7000.00
✅ Donut chart labels — category name + % shown for segments ≥10%; leader lines; overflow:visible
✅ Donut chart enlarged — r=90, sw=28, display 300×225; viewBox 400×300
✅ Calendar amounts black — cal-day-amt and cal-day-panel-total changed from red to #111827
✅ Recurring overlay bug fixed — closeRepeatSheet() now hides overlay; no more frozen screen
✅ Recurring button removed from header — not needed; 🔁 icon inline on recurring transaction rows
✅ Recurring wording removed — rows show 🔁 icon only (inline with note text)
✅ Custom category emoji changed to ⭐ — was 📦 brown box
✅ Repeat frequencies trimmed — only Every Month, Every 3 Months, Every 6 Months, Annually + Nothing
✅ Calculator no longer auto-opens — opens only when Amount tapped; same for category panel
✅ Bottom panel sticky — position:sticky keeps calc/cat panel visible above action bar without scrolling
