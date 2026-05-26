# CONTEXT.md — Expense Tracker

## Current Status
Add page has inline bottom panel: tapping Amount shows calculator, tapping Category shows category grid — both in the same space below the form. Amount defaults to 0.00. Not yet tested end-to-end in Safari. No GitHub Pages deploy yet.

## In Progress
Nothing in progress.

## Up Next
1. Test app fully in Safari — log expenses, test recurring, calendar, stats, calculator, category panel, photo
2. Fix any bugs found during testing
3. Deploy to GitHub Pages (PWA apple-touch-icon needed)

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
