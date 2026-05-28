# CONTEXT.md — Expense Tracker

## Current Status
App deployed to GitHub Pages. Decimal-first amount input and swipe-to-delete (Transactions + Calendar) working and tested on iPhone.

## In Progress
Nothing in progress.

## Up Next
1. Full end-to-end iPhone test — recurring, calendar, stats drill-down, photo, week strip
2. Search / filter transactions

## Backlog
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
✅ Compact add form — Date, Amount, Category row, Note, Description + camera
✅ Category sheet — bottom sheet with custom categories (emoji auto-assigned, color from palette)
✅ Description + photo — textarea, camera button, photo compressed to base64
✅ Full-page add view — view='add'; header back + Save; nav hides
✅ Inline bottom panel — calc and category grid share same panel space; toggle by tapping Amount or Category
✅ Amount defaults to 0.00 — clears on first digit; = and OK always format to 2dp
✅ "Other" hidden from picker — still used as code fallback; "Add" tile opens name-entry sheet
✅ Form row heights fixed — all rows 44px fixed height
✅ Description textarea enlarged — min-height 90px for memo typing
✅ Repeat button icon-only — label removed, 🔁 icon only
✅ Default category changed to blank — "Pick a category" placeholder
✅ Save/Delete bottom action bar — fixed bar at bottom; always visible
✅ Transaction layout restructured — category icon+name on left, note in middle, amount on right
✅ Sort by amount descending — highest spend first within each day group
✅ Category grid 4 columns — tiles smaller and phone-friendly
✅ Category delete rules — only deletable if no transactions tagged
✅ Week strip — horizontal S M T W T F S, swipeable, filters by day
✅ FAB date defaults to selected day — uses txSelDay
✅ Category detail view — tapping a Stats category opens full-page breakdown
✅ Number formatting with commas — rm() uses toLocaleString
✅ Donut chart labels — category name + % for segments ≥10%; leader lines
✅ Recurring overlay bug fixed — closeRepeatSheet() hides overlay correctly
✅ UI polish (Session 8) — font sizes unified to 12px, category label darkened
✅ Month sync fixed — prevTx/nextTx sync txSelDay and txWeekOffset
✅ Future month blocked — > button dims to 30% opacity at current month
✅ Inline + button in Transactions day header
✅ Stats bars and dots removed — fixed-width % and amount columns
✅ Deployed to GitHub Pages — https://rachelhfteoh.github.io/expense-tracker/
✅ PWA apple-touch-icon — gradient circle (purple→pink), white E, dark background
✅ Form font sizes unified — all fields 16px to prevent iOS auto-zoom
✅ iOS zoom fixes — amount field changed to span, date to display span + hidden input, touch-action on calc buttons
✅ Date display fix — fmtFormDate() shows "27 May 2026" format, left-aligned
✅ PWA bottom gap — #app changed to position:fixed; body background set to white to match nav
✅ Nav bar safe area — env() used directly (not via CSS variable) for padding-bottom and height
✅ Swipe-to-delete on transaction rows — swipe left reveals red Delete button; Transactions + Calendar tabs
✅ Decimal-first amount input — cash register style; digits flow right-to-left; dot replaced with 00 button
