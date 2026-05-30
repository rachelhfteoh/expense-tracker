# CONTEXT.md — Expense Tracker

## Current Status
All changes committed and pushed. App live, tested in Safari, in active use with real data.

## In Progress
Nothing in progress.

## Up Next
1. Search / filter transactions

## Backlog
- Budget targets per category
- Category trend history in cat-detail view (monthly breakdown per category)

## Completed — Archive
✅ Requirements gathered — MYR only, expenses only, Sun-Sat calendar
✅ index.html built — Transactions, Calendar, Stats, add/edit/delete, recurring generation on load
✅ CLAUDE.md, CONTEXT.md, MEMORY.md initialised
✅ Initial git commit and GitHub Pages deployment
✅ Colourful theme — gradient background, purple accent, glass cards, pastel stat cards
✅ Calculator keyboard — custom in-app calc; supports +−×÷, cash register style
✅ Compact add form — Date, Amount, Category, Note, Description + camera
✅ Full-page add view — view='add'; header back + Save; nav hides
✅ Inline bottom panel — calc and category grid share same panel space
✅ Week strip — horizontal S M T W T F S, swipeable, filters by day
✅ Category detail view — tapping a category opens full-page breakdown
✅ Swipe-to-delete on transaction rows — Transactions + Calendar tabs
✅ Decimal-first amount input — cash register style
✅ Recurring tab — 4th nav tab; rules with edit sheet and cash register numpad
✅ Smart 🔁 badge — shows only while rule is active
✅ Delete recurring transaction — removes transaction + rule + future instances
✅ iOS fixes — zoom prevention, span amount field, hidden date input, position:fixed app shell
✅ Monthly tab (5th nav) — line chart trend + monthly breakdown by year; tap row → Transactions
✅ Monthly total in Transactions header — shows total for displayed month below month nav
✅ Export/Import backup — settings gear ⚙ in Monthly tab; downloads/restores expenses-backup.json
✅ Recurring tab monthly commitment — shows total monthly committed spend in summary bar
✅ Recurring simplified — frequency reduced to Monthly only
✅ Recurring sub-line — shows category label instead of "since" date
✅ Categories refined — 16 custom categories
✅ Category filter on Monthly tab — filter pill, picker sheet, filtered chart + month list, Peak Month badge
✅ Fix: recurring rule edit now syncs amount to ALL linked transactions (not just today+future)
✅ Monthly tab year totals — yearly total in each year section header
✅ Monthly tab Year/Month toggle — collapse all to year view or expand to month view
✅ Monthly tab cleanup — removed All Time total, Monthly Avg, and month bar graphs
✅ Stats tab renamed to Categories — donut chart removed, replaced with total summary bar
✅ No-cache meta tags — added to reduce stale PWA cache on iOS
