# CONTEXT.md — Expense Tracker

## Current Status
First version of index.html built. Single-file HTML/CSS/JS expense tracker with 3 tabs (Transactions, Calendar, Stats), recurring expenses, 16 categories, MYR currency. Not yet deployed to GitHub Pages.

## In Progress
Nothing in progress.

## Up Next
1. Test the app in Safari — log a few expenses, test recurring, check calendar and stats
2. Set up GitHub repository and deploy to GitHub Pages
3. Add PWA apple-touch-icon (same pattern as habit tracker)

## Backlog
- Search / filter transactions
- Export data (JSON backup)
- Budget targets per category

## Completed — Archive
✅ Requirements gathered — MYR only, expenses only, 3 tabs, Sun-Sat calendar, full repeat options list
✅ index.html built — Transactions, Calendar, Stats views; add/edit/delete; recurring generation on load
✅ CLAUDE.md, CONTEXT.md, MEMORY.md initialised

## Architecture Reference
- Single file: index.html (no build tools)
- localStorage key: 'et_v1'
- Data: transactions[], recurring[]
- render() rebuilds full UI; renderContent() for partial updates
- External deps: Google Fonts Plus Jakarta Sans (CDN)
- Not yet deployed
