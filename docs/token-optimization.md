# Token Optimization

## Why This Matters

Token waste compounds. A session that reads 5 files unnecessarily at startup costs
those tokens on every single session for the life of the project. Over 100 sessions
that is a significant drain on daily limits and a meaningful cost in paid plans.

Three Man Team addresses token waste structurally — the rules are in CLAUDE.md so they
fire automatically, not as a reminder you might follow.

## The Five Rules

Built into every Three Man Team session router:

```
Is this in a skill or memory?   → Trust it. Skip the file read.
Is this speculative?            → Kill the tool call.
Can calls run in parallel?      → Parallelize them.
Output > 20 lines you won't use → Route to subagent.
About to restate what user said → Delete it.
```

## RTK — Bash Output Compression

Token optimizer controls how the team thinks, reads, and responds. RTK handles what
comes back from bash commands.

[RTK](https://github.com/rtk-ai/rtk) is a separate tool that compresses bash command
output before it hits Claude's context window. Tools like `find`, `ls`, and `grep` can
return hundreds of lines Claude doesn't need — RTK intercepts that output and compresses
it, often cutting 50-70% of bash output tokens automatically.

RTK is not part of Three Man Team and we don't manage it. Install it separately if you
use Claude Code CLI heavily. The combination of RTK (bash layer) + token-optimizer
(behavior layer) is where the real savings compound.

Install: https://github.com/rtk-ai/rtk

## Grep Before Read

Never read an entire file to find one thing. Grep for the exact string or line number
first. If you need context around it, use offset and limit on the Read. This single
rule has the highest impact in active build sessions.

## Role-Scoped Loading

Each role loads only what their job requires:
- Architect: checkpoint + BUILD-LOG + ARCHITECT-BRIEF
- Builder: ARCHITECT-BRIEF + relevant reference files only
- Reviewer: REVIEW-REQUEST-[ID] + the diff + specific files Builder listed
- Triage: only the source it was given — no project files at all

The full project spec, schema, and flow docs stay on disk until explicitly needed.

## Checkpoint-First

SESSION-CHECKPOINT.md replaces reading BUILD-LOG and the full spec when it is current.
Architect writes it at the end of every session. It should cover: where we stopped,
what was decided, what is open, and the exact resume prompt.

## Cost Scales With Turns × Context

In long agent runs most of the bill is the context being re-read on every turn (cache reads).
A resumed agent drags every earlier round along — measured on a real project, a Builder resumed
four times cost about 6× a fresh one for the same work. Start a fresh Builder and Reviewer for
every round, and stop a Builder that passes ~80 tool calls so a fresh one can take over.

## One Round, One File

Review requests and feedback are written per round (`REVIEW-FEEDBACK-3-r2.md`), never appended.
Files that only grow get re-read by every later agent. Finished steps move to `handoff/archive/`
at the Deploy Gate. The checkpoint stays under 80 lines.

## Triage on Haiku

Long logs, test output and search sweeps go to the Triage role on Haiku with low effort. It writes
a summary of at most 40 lines; the other roles read that, not the raw output.

## Effort Where It Is Cheap

Raise effort on short, high-leverage runs — the Reviewer. Extra thinking there costs little and
catches problems a round earlier. Builder runs are long and every extra turn re-reads a large
context, so the Architect sets Builder effort per round in the brief: `medium` for mechanical
rounds, `high` for logic rounds.

## Architect Context Hygiene

The Architect proposes a fresh session after every Deploy Gate and `/compact` before long rounds.
Its context is rebuilt in full whenever the prompt cache expires — every wait on a long background
run — so a lean Architect is the cheapest Architect.

## CLAUDE.md as Router

CLAUDE.md should never exceed ~50 lines. It is a routing file, not a knowledge base.
Every byte in CLAUDE.md costs tokens on every session. Keep it lean.
