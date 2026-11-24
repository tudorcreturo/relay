# Portfolio site

**Goal:** a portfolio page with projects grouped by category and a full-screen gallery.

## Decisions
- 11/23 18:40 Static site, no framework (provisional: revisit only if the gallery needs it)
- 11/23 18:42 Categories: web, print, photo (provisional)

## Standing agreements
- Local commits are fine. Nothing is deployed without an explicit request in that round.

## Slices

| id | what | model | files | done when | status |
|---|---|---|---|---|---|
| 1 | page skeleton and project grid | sonnet | index.html, css/ | grid renders 6 placeholder cards at 3 widths | done |
| 2 | category filter | sonnet | js/filter.js | clicking a category shows only its cards | doing |
| 3 | full-screen gallery | opus | js/gallery.js | arrow keys and Esc work, no scroll jump on close | todo |
| 4 | real content and optimized images | sonnet | content/, img/ | all texts in, images under 200 KB | todo |
| 5 | deploy | sonnet | none | live URL loads, checked on phone | todo |
