---
name: adversarial-review
description: Runs a ruthless, severity-ranked review that assumes the artifact is broken until proven otherwise, then returns findings + a binary ready/not-ready verdict. Use proactively before approving, locking, shipping, merging, or finalizing ANY artifact (spec, plan, design, architecture, code, PR, doc, config, contract), after finishing each task/phase of a plan, and whenever tempted to say "looks good" / "this is ready" / "good enough" - even if the user did not explicitly ask for a review. If something is about to be signed off, use this first.
---

# Adversarial Review

## Overview

A ruthless review pass that assumes the artifact is flawed until proven otherwise. The job is to find what is wrong BEFORE it costs real time or money downstream. Flattery is failure. Works on anything reviewable: specs, plans, designs, architecture, code, PRs, docs, configs, contracts.

**Core principle: you did not write it and you have no ego in it. Default assumption is that it is broken.**

## When to Use

- Before approving, locking, shipping, merging, or finalizing ANYTHING.
- The moment you catch yourself thinking "looks good", "this is ready", or "good enough".
- After finishing each task/phase of a plan, before committing or moving on.
- When someone asks "is this done?" / "any problems?" / "review this".
- Whenever a first draft is about to be accepted as final.

Do NOT skip because the artifact "seems simple" - simple artifacts hide unexamined assumptions, which is where the expensive failures live.

## The Iron Rule

**Never return SHIP IT on the first pass unless the artifact is genuinely airtight.** Most first passes end in NOT READY. If you found nothing, you did not look hard enough - look again before concluding.

**Violating the letter of this rule is violating its spirit.** "It's basically fine" is not a verdict.

## Ground Every Claim

Review the artifact as the single source of truth a builder would implement from. But verify its claims against reality where you can: read the referenced code, check the cited file/line, confirm the API exists, run the numbers. A finding grounded in "I checked, the function is actually named X" beats a vague "this might be wrong."

## Attack Across These Axes

1. **Holes** - What is missing? Unstated assumptions, undefined terms, unhandled cases, absent error/edge/failure behavior, missing acceptance criteria.
2. **Logic bugs** - Contradictions, circular dependencies, steps that cannot work as described, ordering problems, race conditions, wrong data flow.
3. **Ambiguity** - Anything a builder could interpret two ways. Name each one.
4. **Scope & feasibility** - Over-engineering, under-specification, hidden complexity, unrealistic constraints, costs that will balloon.
5. **Hard questions** - The uncomfortable questions whose answers might invalidate the whole approach. Ask them. Do not soften them.

## Output Format

```
## Findings
(ordered by severity, blockers first. Each finding is one block:)
- **Severity:** Blocker | Major | Minor
- **What:** the problem in one sentence
- **Where:** the section / file / line it lives in
- **Why it matters:** the concrete downstream consequence
- **Fix:** a specific, actionable correction (NOT "consider improving")

## Hard Questions
(numbered; the questions that must be answered before this is buildable/shippable)

## Verdict
(exactly one:)
- NOT READY - at least one Blocker or Major, OR any open hard question. (Default.)
- SHIP IT AS IS - genuinely nothing left worth changing: no Blockers, no Majors,
  no unanswered hard questions, remaining Minors not worth another revision.
```

## Severity Definitions

| Severity | Meaning |
|----------|---------|
| **Blocker** | Wrong/contradictory/unbuildable as written. Cannot proceed. |
| **Major** | Will cause real downstream damage (cost, rework, incorrect behavior) if unfixed. |
| **Minor** | Real but localized; worth fixing, does not reopen the design. |

## The Loop (review -> revise -> re-review)

One pass is rarely enough. Run the cycle until it converges:

1. Review -> findings + verdict.
2. If NOT READY: the author (you or another agent) fixes the findings.
3. Re-review the revised artifact - and specifically hunt for issues the FIX introduced (fixes create new seams).
4. Repeat until SHIP IT.

Track convergence honestly: "pass 1: 4 blockers -> pass 2: 2 majors -> pass 3: 1 seam -> pass 4: clean" is a healthy trajectory. If severity is not dropping, the fixes are not real.

## Rules (non-negotiable)

- Do NOT fix or rewrite the artifact in a review pass. Review it. (Fixing is a separate step.)
- Do NOT invent praise to balance criticism. No "great work, but...".
- Do NOT manufacture filler findings to look thorough. If a pass is genuinely clean, say so and give SHIP IT.
- Tempted to say "looks good"? Look harder first - find the holes, then decide.

## Red Flags - you are rationalizing, STOP

| Thought | Reality |
|---------|---------|
| "This is too simple to need review" | Simple = unexamined assumptions. Review it. |
| "Looks good to me" | You have not attacked it yet. Run the axes. |
| "I'll just approve to keep momentum" | A downstream blocker costs more than a review pass. |
| "I don't want to be harsh" | Flattery is failure. The artifact is the target, not the author. |
| "SHIP IT on the first pass" | Almost never true. Look again. |
| "I found a few things, good enough" | Order by severity and finish the sweep before the verdict. |
| "Re-review is overkill after a fix" | Fixes introduce new seams. Re-review hunts exactly those. |

## Common Mistakes

- **Stopping at the first issue.** Sweep all five axes before concluding.
- **Vague fixes.** "Add error handling" is not a fix. Name the case and the correction.
- **Reviewing only what is written, never reality.** Verify cited code/APIs/numbers.
- **Softening hard questions.** The ones that might invalidate the approach are the point.
- **Declaring victory early.** Severity must actually reach zero, not "close enough".
