<p align="center"><img src="./poirot.svg" alt="Poirot" width="180"></p>

<h1 align="center">Poirot</h1>

<p align="center">A Claude Code skill that turns bug investigation into a disciplined method instead of a guess.</p>

## The problem it solves

Claude Code (and any capable model) is already good at *reasoning* about code. What it lacks under pressure is patience. Its default debugging failure mode is impatience: pattern-match a symptom to a familiar cause, edit the file, declare victory — without ever reproducing the bug or proving that change was responsible. You end up with a "fix" for a bug that was never confirmed, in a working tree where probe edits and fix edits are tangled together.

Poirot is a behavioral leash against that. It doesn't make the model smarter; it stops it from skipping the steps that make an investigation trustworthy.

## What it enforces

- **Reproduce before theorizing.** No hypothesis until there's a deterministic failing case.
- **Read-only until the root cause is confirmed.** Instrumentation is allowed during investigation but must be tracked and reverted — so probe edits never contaminate the fix.
- **Differential diagnosis with active disconfirmation.** List ranked candidate causes, then try to *disprove* the top one rather than collect evidence that flatters it.
- **Git forensics.** `git blame`, `git log -S/-G`, and `git bisect` — the history tools that usually find a regression faster than reading code, and that the model habitually skips.
- **Proximate vs. root cause.** Fix the root, with the Phase 1 reproduction as the test that proves it.

## When to use it

Use it for a **specific reported symptom or regression** — "X is broken," "this worked last week," "Y returns the wrong value," "intermittent failure in Z." It's deliberately *not* for open-ended "find any bugs" sweeps; those produce noise, and tests/types/linters serve them better.

## Install

Global (available in every project):

```bash
git clone https://github.com/dharmabumzee/claude-poirot ~/.claude/skills/poirot
```

Or per-project — copy `skills/poirot/SKILL.md` into `.claude/skills/poirot/SKILL.md` in your repo.

Then just describe the bug. Claude Code triggers the skill from the symptom.

## License

MIT — see [LICENSE](./LICENSE).