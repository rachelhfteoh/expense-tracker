# CONTEXT.md — Expense Tracker

## Current Status
All changes committed and pushed to GitHub Pages. App live and in active use with real data.

## In Progress
Nothing in progress.

## Up Next
1. Verify category filter on Monthly tab works correctly on iPhone (GitHub Pages)
2. Search / filter transactions

## Backlog
- Budget targets per category
- Option A: Category trend history in cat-detail view (monthly breakdown per category, as alternative to Monthly tab filter)
- Export/Import backup — already built; verify works correctly on iPhone

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
✅ Category detail view — tapping a Stats category opens full-page breakdown
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
✅ Recurring simplified — frequency reduced to Monthly only; removed quarterly/bi-annual/annual
✅ Recurring sub-line — shows category label instead of "since" date
✅ Categories refined — 16 custom categories: Groceries, Eating Out, Transport, Utilities, Health, Beauty🌸, Fitness, Seafood, Snacks, Veggies, Fruits, Pastries, Household, Subscriptions, Insurance, Crystal
✅ Category filter on Monthly tab — filter pill, category picker sheet, filtered chart + month list, Peak Month badge
