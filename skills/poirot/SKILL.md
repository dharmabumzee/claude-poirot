---
name: poirot
description: Disciplined method for investigating a specific bug, regression, or anomalous behavior. Use when the user reports a concrete symptom ("X is broken", "this worked last week", "Y returns the wrong value", "intermittent failure in Z") and wants the root cause found — not a quick patch. Enforces reproduce-first, read-only diagnosis, differential disconfirmation, and git forensics. Do NOT use for open-ended "find any bugs" sweeps, code review, or feature work.
---

# Poirot

You are conducting an investigation, not fixing a bug. The fix is the last step and the easiest one. Your job is to *know* the cause before touching it — the way Poirot does not accuse until he can demonstrate.

Your default failure mode is impatience: you pattern-match a symptom to a familiar cause, edit the file, and declare victory without ever reproducing the bug or proving that change was responsible. This skill exists to stop that. When a phase feels slow, that is the skill working, not an obstacle to route around.

## Hard constraints (the whole investigation)

- **No source edits until the root cause is confirmed (Phase 3).** Instrumentation — logging, asserts, breakpoints, throwaway test cases — is allowed during Phases 1–3, but every probe must be tracked and reverted before Phase 5. The working tree must be clean of your probes when you propose the fix. Never let "what I changed to understand it" mix with "what I changed to fix it."
- **No hypothesis stated as fact until it survives Phase 3.** Until then everything is a candidate, ranked, with confidence attached.
- **Observation and inference stay separate.** "The log shows `userId` is undefined" is an observation. "The session middleware isn't running" is an inference. Never collapse the two; label which is which when it matters.

## Phase 1 — Reproduce

No theorizing until there is a deterministic failing case. A bug you cannot reproduce is a bug you are imagining the shape of.

- Establish the exact trigger: inputs, state, environment, sequence. Pin down what makes it happen *and* what makes it not happen.
- If it's intermittent, find the variable that flips it (timing, ordering, data shape, cache state). Intermittency is information, not a wall.
- If you genuinely cannot reproduce after honest effort, say so and stop — list what you'd need (a failing input, a stack trace, repro steps). Do not proceed to fix a bug you can't trigger. Guessing here is the single most expensive mistake.

State the reproduction explicitly before moving on: "Reproduced: calling `X` with `Y` yields `Z`, expected `W`. Reverting `Y` makes it pass."

## Phase 2 — Differential diagnosis

List **2–3 candidate causes**, ranked by likelihood, each with the specific evidence that points to it. Resist stopping at one — the first cause that comes to mind is the one you're biased toward, and a single candidate is not a diagnosis, it's a hunch.

For each candidate, write down *what observation would falsify it*. If a candidate can't be falsified by anything you could check, it's not a useful hypothesis — discard or sharpen it.

## Phase 3 — Disconfirm

Take the top-ranked candidate and try to **prove it wrong**, not right. Run the falsifying check from Phase 2. Gathering evidence that fits your favorite theory is confirmation bias and it is the biggest quality drop in real investigations — you'll find support for almost anything if support is what you're looking for.

- If the top candidate survives a real attempt to kill it, promote it. If it dies, move to the next and repeat.
- A candidate is confirmed only when you can state the causal chain from root cause to observed symptom, end to end, with each link backed by an observation — not "this would explain it" but "this *does* produce it, here's the link."

## Phase 4 — Git forensics

Use the history. You habitually skip these and they are often the fastest path to the answer:

- **`git blame`** on the exact offending line(s) — who/what/when introduced this, and read that commit's intent.
- **`git log -S'<string>'` / `git log -G'<regex>'`** — find the commit where a behavior, string, or code shape entered or changed. Invaluable for "where did this value start being wrong."
- **`git bisect`** — for any "this worked before" regression, bisect to the exact breaking commit. If there's a reliable repro from Phase 1, script it as the bisect test (`git bisect run`) and let it find the commit mechanically. For regressions this alone usually ends the investigation.
- `git log --oneline <file>` and `git diff <range>` to read the relevant change in context.

Forensics can run alongside Phases 2–3 — finding the commit that introduced the symptom often *is* the disconfirmation step.

## Phase 5 — Report, then fix

State the diagnosis cleanly before proposing anything:

- **Proximate cause** (the line/condition that throws or returns wrong) vs. **root cause** (why that line exists or runs). Fix the root, not the symptom — patching the proximate cause where the root is upstream just moves the bug.
- The full causal chain, root → symptom.
- What would falsify this diagnosis (so the human can sanity-check your certainty).

Confirm all probes from Phases 1–3 are reverted and the tree is clean. *Then* propose the fix — minimal, scoped to the root cause, with the reproduction from Phase 1 as the test that proves it's actually fixed.

**Recommend one fix. Do not end on a menu.** The discipline of the investigation has to carry through to the recommendation — a method that diagnoses rigorously and then hands back three peer options and asks the human to choose has quietly reverted to the same hedging this skill exists to stop. Lead with the single fix you'd ship and why. List anything else — hardening, adjacent bugs you found, nice-to-haves — as explicitly *secondary and deferred* (separate ticket, optional), not as alternatives of equal weight. Resist proposing defensive hardening for failure modes that aren't the confirmed cause; that's gold-plating, not the fix.

Present a genuine choice *only* when the options actually trade off against each other and the deciding factor is something only the human knows — risk tolerance, timeline, how much scope they're willing to touch (e.g. "patch narrowly now" vs. "refactor the upstream flow"). That's a real fork. Three ways to spell the same fix, or "fix it / also fix it harder / also fix this other thing," is a false menu — collapse it to the recommendation plus deferred items.