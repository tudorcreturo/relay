# relay

A Claude Code skill for building software across many sessions and many days, on a limited subscription, without burning your usage on lost context.

Each session runs one small slice of the project, verifies it, and passes the baton to the next session through three small files on disk. Like a relay race: no single runner goes the whole distance, and the baton never drops.

**Keywords:** Claude Code skill, agent skills, multi-session development, context management, usage limits, Pro plan, handoff, long-running projects, incremental development.

## Why it exists

This came out of three days of building a native macOS CAD app with Claude Code (Swift, Metal, OpenCASCADE, 4,500+ automated tests) on a subscription with a 5-hour usage window. The costly lessons:

| What happened | What it cost | The rule that came out of it |
|---|---|---|
| Sessions left unfinished when the limit hit, resumed after the reset with 600-800k tokens of history | 20-30% of the 5-hour window just to reload context, before any new work | Hard stop at 80%, with a resume note; come back in a new chat |
| `/compact` on a large context | ~12% of the window | Open a new chat instead of compacting |
| 7 agents started at once (4 Opus, 3 Sonnet) | ~45% of the window in 10 minutes, mostly on startup reading | One worker at a time, plus at most one cheaper reviewer |
| Tests passing on wrong geometry | A redone slice | "Done" means verified by something that actually sees the result |
| Intermediate results kept only in a temp folder | Lost on reboot | Anything expensive to redo is saved in the project |

The pace that worked in the end: 40-50 minutes of focused work per usage window, on one well-cut, achievable slice that adds a real piece to the finished app.

## How usage is actually spent

1. **Every message resends the whole conversation.** In a 600k-token chat, a one-line question costs 600k tokens of reading.
2. **The prompt cache lasts about an hour on a subscription.** Come back later, or after a usage reset, and the first message rereads the whole history at full price.
3. **`/compact` is not free.** To summarize the conversation, Claude has to read all of it.
4. **Every new session or subagent pays its own startup:** project instructions, memory, tools. Start helpers in the project folder, with the cheapest model that can do the job.

So: small context is cheap, large context is expensive on every single message. relay keeps every session small.

## How it works

The state lives inside your project, in `plan/<project>/`:

| File | Holds | Size |
|---|---|---|
| `PLAN.md` | goal, your decisions, standing agreements, the slice table | a page or two |
| `STATE.md` | where you stopped, current slice, exact next step, how to resume | under 30 lines |
| `LOG.md` | one short entry per session, newest first | grows, but only the last 2 entries are read |

Each session follows the same cycle:

1. **Read the state:** only `STATE.md`, the slice table, and the last 2 log entries.
2. **Startup check:** build and quick tests (or load the site locally), to catch anything the previous session left broken.
3. **One slice:** one visible result, a checkable "done when", the files it may touch, the right model, 40-50 minutes.
4. **Verify** against "done when".
5. **Close:** commit locally (if you agreed), update the three files, name the session.
6. **Resume choice, made for you:** under ~150k tokens and a short break → continue in the same chat while the cache is warm; otherwise → a new chat with `/relay`.

Hard stop at 80% of the usage window. You don't need to run to 100%: the remaining 15-20% stays free for other productive sessions.

## Choosing the model

| Model | For |
|---|---|
| Opus | new code, algorithms, complex interactions, hard bugs, the initial plan of a large project |
| Sonnet | most of the work: features with a clear spec, tests, reviews, integrations |
| Haiku | mechanical work: checking links, renaming, scanning for missing items |

Subagents always get their model set explicitly. One worker at a time.

## Install

Personal (available in every project):

```bash
git clone https://github.com/tudorcreturo/relay.git ~/.claude/skills/relay
```

Or for a single project: copy `SKILL.md` to `.claude/skills/relay/SKILL.md` inside that project.

## Use

| You type | What happens |
|---|---|
| `/relay` + an idea | a new plan: Claude sizes the idea, and if it needs more than one session, splits it into slices |
| `/relay` | resumes the project exactly where the last session stopped |
| "close the session" | verify, save, write the resume note, tell you how to continue |

Example: `/relay Add a portfolio page with projects grouped by category and a full-screen gallery.`

## Tips that are easy to miss

- Keep `CLAUDE.md` short (under ~200 lines); it is loaded on every turn of every session.
- `/usage` shows where your tokens go (skills, subagents, tools); `/context` shows what fills the current context.
- Two failed corrections in a row → new chat with a better prompt that includes what you learned.
- `/rewind` (or Esc twice) rolls back conversation and files, so trying the bold option first is cheap.
- For small product details, let Claude make a provisional decision and validate it by testing the result, instead of answering a long questionnaire up front.
- Write standing agreements (like "local commits are fine") into `PLAN.md`, so you are not asked every session.

## Background reading

- Anthropic, [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents): progress file, feature list, one feature per session, startup check
- Claude Code docs, [Best practices](https://code.claude.com/docs/en/best-practices) and [Manage costs effectively](https://code.claude.com/docs/en/costs)

## License

MIT. Made by Tudor Crețu, with Claude Code. Issues and improvements are welcome.
