# Three Man Team — Session Router

## Token Rules — Always Active

```
Is this in a skill or memory?   → Trust it. Skip the file read.
Is this speculative?            → Kill the tool call.
Can calls run in parallel?      → Parallelize them.
Output > 20 lines you won't use → Route to subagent.
About to restate what user said → Delete it.
```

Grep before Read. Never read a whole file to find one thing.
Do not re-read files already in context this session.

---

## Session Start — Every Role

1. Load your token-optimizer skill if you have one — first, before anything else.
2. Check `SESSION-CHECKPOINT.md` — if active and recent, read it. That is your state.
3. Load your role file — copied to project root → `ARCHITECT.md` · `BUILDER.md` · `REVIEWER.md` · `TRIAGE.md`
4. If no checkpoint — Architect reads `BUILD-LOG.md` + `ARCHITECT-BRIEF.md` only.

**Project Owner role is set by the human. Do not ask.**

---

## Reference Files — On Demand Only

| File | Load when |
|---|---|
| Project spec | Architect needs it; checkpoint doesn't cover it |
| ARCHITECT-BRIEF.md | Builder and Reviewer load at task start |
| BUILD-LOG.md | Architect checks status; Builder updates when done |
| REVIEW-REQUEST-[ID].md | Reviewer loads its own round's file at review start |
| REVIEW-FEEDBACK-[ID].md | Builder loads the file Architect names for the fix round |
| SPECIALIST-REVIEW-[ID].md | Optional domain reviewer's verdict; Architect checks it at the Deploy Gate |
| triage/[ID]-[topic].md | Whoever spun up Triage reads the summary instead of the raw output |
| decisions/D-[NN].md | Architect writes; Advisor reads and answers in the same file |
| archive/ | Only when a question needs history — grep, never read whole |

One round, one file. Nobody appends to an earlier round's file.

