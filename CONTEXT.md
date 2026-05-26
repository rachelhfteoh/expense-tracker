# CONTEXT.md — Expense Tracker

## Current Status
Add/edit form rebuilt as compact full-page view with persistent calculator. Category picker is a separate bottom sheet. Custom categories, description field, and photo capture added. Not yet tested end-to-end. No GitHub remote set up yet.

## In Progress
Nothing in progress.

## Up Next
1. Test app fully in Safari — log expenses, test recurring, calendar, stats, calculator, category sheet, photo
2. Fix any bugs found during testing
3. Set up GitHub remote and push
4. Deploy to GitHub Pages (PWA apple-touch-icon needed)

## Backlog
- Search / filter transactions
- Export data (JSON backup)
- Budget targets per category

## Completed — Archive
✅ Requirements gathered — MYR only, expenses only, Sun-Sat calendar, 14 repeat frequencies
✅ index.html built — Transactions, Calendar, Stats, add/edit/delete, recurring generation on load
✅ CLAUDE.md, CONTEXT.md, MEMORY.md initialised
✅ Initial git commit made (no remote yet)
✅ Colourful theme — gradient background, purple accent, glass cards, pastel stat cards (matches habit tracker)
✅ Design polish — SVG icons for FAB/close buttons, larger touch targets, nav indicator line, transitions
✅ Calculator keyboard — custom in-app calc slides up on amount tap; supports +−×÷, preview, OK to confirm
✅ Compact add form — Date, Amount, Category row, Note, Description + camera icon (no category grid on screen)
✅ Category sheet — slides up on category tap; pencil icon to add custom categories (emoji, name, colour swatch)
✅ Description + photo — textarea below note, camera button opens Camera/Gallery action sheet, photo compressed and stored
✅ Full-page add view — tapping + navigates to view='add'; header has back button + Save; calc auto-open at bottom; nav hides
