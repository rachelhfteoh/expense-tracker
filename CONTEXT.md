# CONTEXT.md — Expense Tracker

## Current Status
App is functional with polished form UI, restructured transaction layout, week strip on transactions tab, and category delete rules. Tested manually in Safari with real data. No GitHub Pages deploy yet.

## In Progress
Nothing in progress.

## Up Next
1. Test all flows end-to-end in Safari — recurring, calendar, stats, photo, week strip navigation
2. Deploy to GitHub Pages (PWA apple-touch-icon needed)

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
✅ Default category changed to Groceries — was "Other" (hidden fallback), now first real category
✅ Save/Delete bottom action bar — fixed bar at bottom; Save left, Delete right; both always visible
✅ Transaction layout restructured — category icon+name on left, note in middle, amount on right
✅ Sort by amount descending — highest spend first within each day group (Transactions + Calendar)
✅ Category grid 4 columns — was 3 columns; tiles smaller and phone-friendly
✅ Category delete rules — only deletable if no transactions tagged; built-in + custom both respect this
✅ Week strip on Transactions tab — S M T W T F S, today purple, expense dots, swipeable, tapping day scrolls to group
