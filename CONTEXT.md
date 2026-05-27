# CONTEXT.md — Expense Tracker

## Current Status
App is functional and polished. UI cleanup session complete — font sizes unified, month sync fixed, future month navigation blocked, stats bars removed, dots removed, FAB repositioned.

## In Progress
Nothing in progress.

## Up Next
1. Deploy to GitHub Pages (PWA apple-touch-icon needed)
2. End-to-end test on actual iPhone — recurring, calendar, stats drill-down, photo, week strip

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
✅ Default category changed to blank — was Groceries; now blank with "Pick a category" placeholder
✅ Save/Delete bottom action bar — fixed bar at bottom; Save left, Delete right; both always visible
✅ Transaction layout restructured — category icon+name on left, note in middle, amount on right
✅ Sort by amount descending — highest spend first within each day group (Transactions + Calendar)
✅ Category grid 4 columns — was 3 columns; tiles smaller and phone-friendly
✅ Category delete rules — only deletable if no transactions tagged; built-in + custom both respect this
✅ Week strip fixed — was vertical; now horizontal S M T W T F S with class="week-strip"
✅ Day filtering on Transactions tab — tapping a strip day shows only that day's transactions
✅ FAB date defaults to selected day — uses txSelDay not todayStr() when opening from Transactions
✅ Category detail view — tapping a Stats category opens full-page breakdown (view='cat-detail')
✅ Number formatting with commas — rm() uses toLocaleString; RM 7,000.00 not RM 7000.00
✅ Donut chart labels — category name + % for segments ≥10%; leader lines; overflow:visible
✅ Recurring overlay bug fixed — closeRepeatSheet() now hides overlay correctly
✅ UI polish (Session 8) — font sizes unified to 12px, category label darkened to #6b7280
✅ Month sync fixed — prevTx/nextTx now update txSelDay and txWeekOffset via syncTxToMonth()
✅ Future month blocked — nextTx/nextCal/nextSt blocked at current month; > button dims to 30% opacity
✅ FAB repositioned — 50px size, right: calc(24px + env(safe-area-inset-right)); hidden in Transactions/Stats
✅ Inline + button in Transactions — replaces FAB; always shown in day header even when empty
✅ Stats bars removed — progress bars removed from category list; % and amount columns fixed-width aligned
✅ Stats dots removed — coloured dots removed from category list rows; emoji sufficient identifier
