---
name: relay
description: Plan and resume software projects that span many Claude Code sessions (a website, an app) without wasting context or usage limits. Each session runs one small slice and passes the baton through files on disk. Use when the user types /relay (with or without an idea), or asks to "continue the project", "pick up where we left off", "plan this across sessions", or "close the session".
---

# /relay: multi-session projects in small slices

Goal: every session starts small, does one slice, and closes cleanly. The state lives on disk, inside the project, in `plan/<slug>/`:

- `PLAN.md`: the goal, the user's decisions (dated), standing agreements (for example "local commits are fine"), and the slice table (id, what, model, files touched, done when, status)
- `STATE.md`: where we stopped, the current slice, the exact next step; under 30 lines
- `LOG.md`: one short entry per session, newest first

Pick the mode from what you find.

## Session title

Every `/relay` session gets a title in one format, so sessions are easy to find and sort: `relay MM/DD HH:MM <slice>`, local time, 24-hour clock (`date "+%m/%d %H:%M"`). Examples: `relay 11/23 18:51 1b.3a`, `relay 11/23 18:51 gallery fullscreen`.

- `<slice>`: the slice id from `PLAN.md` if there is one, otherwise 2-3 words on what the session does (`plan portfolio` while planning). Before a slice is chosen: `relay 11/23 18:51 state`.
- Set it as soon as the slice is chosen: `/rename` in the terminal, or the app's session-title tool if one is available. Keep the start time; if the slice changes, change only the last part.
- Helper sessions you start get the same format, with their own slice id.

## A. New idea (`/relay` plus an idea, no plan for it yet)

1. Ask at most 3 questions, only the ones that change the result. Decide the rest yourself, pick the most sensible option, and mark it "provisional" in `PLAN.md`.
2. Read only the project files that matter for the idea. For large files, search first (`grep`), then read just the relevant range.
3. Size it:
   - fits in one session (under ~45 minutes, a few files): say so and just do it, no plan files;
   - does not fit: write `plan/<slug>/PLAN.md`, `STATE.md` and `LOG.md`.
4. Slices: each has one visible result, a checkable "done when" (preferably in the browser or with a test), the files it touches and the ones it must not touch, the recommended model, and 40-50 minutes of work. Deploying or publishing is always its own, final slice.
5. Show the plan in at most 15 lines and wait for approval. If planning took a lot of context, recommend starting the first slice in a new chat.

## B. Resume (`/relay` with no idea, or "continue the project")

1. Cheap resume first: if `STATE.md` has `resume: continue_here` and that session closed less than 50 minutes ago, tell the user to go back to that chat (its history is still cached) and stop here.
2. If `plan/` holds several projects, ask which one.
3. Read only: `STATE.md`, the slice table in `PLAN.md`, the last 2 entries of `LOG.md`, `git log --oneline -5` and `git status` (if the project uses git). Nothing else in full.
4. Startup check: build and run the quick tests, or load the site locally, to catch anything the previous session left broken. If the project has a check script (for example `scripts/check.sh`), run that. If it fails, fixing it is today's slice.
5. Set the session title (see "Session title"), then report in 3 lines: where we are, today's slice, the recommended model. If the session runs on a different model than the slice needs, say so.
6. Work on one slice only. If it turns out bigger, split it in `PLAN.md` and finish only the first part.

## C. Close ("close the session", the slice is done, or usage reaches 80%)

Hard stop at 80% of the usage window (5-hour or weekly). There is no need to run to 100%: the remaining 15-20% stays free for other sessions, and a session closed cleanly never has to be resumed with its whole context.

1. Verify the slice against its "done when".
2. Save: commit locally only if `PLAN.md` holds that agreement or the user gives it now; otherwise list what the commit would contain.
3. Update the slice status in `PLAN.md`, write the exact next step in `STATE.md`, add a 3-5 line entry to `LOG.md`.
4. Check that the session title follows the format and names the last slice worked on.
5. Choose how to resume and write it in `STATE.md` as `resume:`, with the time:
   - `continue_here`: the context is under ~150,000 tokens and this is a short break, not a usage limit. Say: "Continue in this chat" (from a terminal: `claude --resume "<session title>"`).
   - `new_chat`: in every other case, including after a usage reset. Say: "Open a new chat and type /relay".
6. The last line of the message is that choice, so the user has nothing to decide.

## Helpers: subagents and parallel sessions

- One worker at a time, plus at most one reviewer on a cheaper model. Parallel agents look faster, but each one pays its own startup reading and they drain a small plan quickly.
- Always set the model explicitly, never inherit it: Opus for new code, algorithms and hard bugs; Sonnet for most work, tests and reviews; Haiku for mechanical reads and checks.
- A helper starts in the project folder, never in a large unrelated workspace whose instructions and tools it would load and pay for on every turn.
- Its brief lives on disk; its prompt only points to it. Its result is written to disk and verified by you before it counts as done. When it has delivered, it stops.

## Rules

- Nothing gets published (deploy, upload, send) without the user's explicit request in that round.
- No `/compact` on a large context; open a new chat instead.
- Anything expensive to redo (extractions, long reports) is saved in the project, never only in a temporary folder.
- Long output (builds, tests) goes to files; bring only the tail or a summary into the chat.
- Write the user's decisions into `PLAN.md` the moment they are made.
