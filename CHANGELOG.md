# Changelog

## Unreleased
- Session lock (`.active`), so two chats do not work on the same project at once.
- Stale-state check on resume: `head:` hash in `STATE.md` against current git.
- `est` and `real` minutes and usage % in `LOG.md`, so estimates come from data.
- Slice statuses, a two-failed-attempts rule, a parking lot, a rollback point per slice.
- Helper recipe (cwd, fresh context, 5-line prompt) and a `/context` check on the first helper.
- `examples/portfolio/` with filled-in PLAN, STATE and LOG files.

## 2026-11-23
- First answer states the slice and a realistic estimate, once.
- Session titles: `relay MM/DD HH:MM <slice>`.
- Initial release.
