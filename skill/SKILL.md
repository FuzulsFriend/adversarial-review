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

**Never return SHIP IT on the first pass unless the artifact is genuinely airtight.** Most first passes end in NOT READY. If you found nothing, look again for missed failure modes; do not invent findings to fill a quota.

**Violating the letter of this rule is violating its spirit.** "It's basically fine" is not a verdict.

## Ground Every Claim

Review the artifact as the single source of truth a builder would implement from. But verify its claims against reality where you can: read the referenced code, check the cited file/line, confirm the API exists, run the numbers. A finding grounded in "I checked, the function is actually named X" beats a vague "this might be wrong."

**Seed reviewers to go deep, not wide-and-shallow.** When facts are already verified, hand them to the reviewer(s) up front: "these are confirmed - do NOT re-flag them; go deeper or find NEW ones." This spends the pass on depth - the non-obvious failure modes - instead of re-discovering surface issues already known. (A pass that just re-reports what you already knew was wasted.)

## Attack Across These Axes

1. **Holes** - What is missing? Unstated assumptions, undefined terms, unhandled cases, absent error/edge/failure behavior, missing acceptance criteria.
2. **Logic bugs** - Contradictions, circular dependencies, steps that cannot work as described, ordering problems, race conditions, wrong data flow.
3. **Ambiguity** - Anything a builder could interpret two ways. Name each one.
4. **Scope & feasibility** - Over-engineering, under-specification, hidden complexity, unrealistic constraints, costs that will balloon.
5. **Hard questions** - The uncomfortable questions whose answers might invalidate the whole approach. Ask them. Do not soften them.

## Scaling the Review: Solo vs Fan-Out

Match the review's breadth to the artifact's size and stakes.

- **Small / low-stakes:** one reviewer runs all five axes in a single pass.
- **Large / high-stakes** (a spec, an architecture, a PR touching many files): run **N reviewers in PARALLEL, each assigned ONE distinct lens** (e.g. correctness, security, ops/failure-modes, spec-vs-reality, cost/scope), blind to each other. Then a **synthesis pass** dedupes, merges overlaps, and severity-ranks.
- **Cross-lens consensus is a confidence signal** a solo pass cannot produce: "3 of 4 lenses independently flagged this" is a stronger Blocker than one reviewer's hunch. Note the consensus in the finding.

Fan-out finds more because each lens is blind to the others - no reviewer rationalizes "someone else probably covered that." Keep the lenses genuinely distinct, or they collapse into one redundant pass.

## Output Format

```
## Findings
(ordered by severity, blockers first. Each finding is one block:)
- **Severity:** Blocker | Major | Minor
- **What:** the problem in one sentence
- **Where:** the section / file / line it lives in
- **Proof:** the failing test, or the file:line you read. Without proof it is a Hard Question, not a Blocker or Major.
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
2. **Triage before fixing.** A long findings list is NOT a to-do list. Sort every finding into **fix-now** (real Blocker/Major), **fold-in** (cheap, in code you are already touching), or **defer-and-document** (real but the fix costs more than the risk it carries). Treating every finding as a mandate is how a review *causes* over-engineering. For each deferral, name why it is acceptable.
3. If NOT READY: the author (you or another agent) fixes the fix-now + fold-in findings.
4. Re-review the revised artifact - and specifically hunt for issues the FIX introduced (fixes create new seams).
5. Repeat until SHIP IT.

**Inside a lead-loop program**, its exit rule replaces this loop: stop when no proven Blocker or Major remains, after at most 2 attacker rounds. Hard questions go to the decisions list, not into the verdict. Refuters only for Blockers and Majors in code reviews. Verdict words: READY / NOT READY.

Track convergence honestly: "pass 1: 4 blockers -> pass 2: 2 majors -> pass 3: 1 seam -> pass 4: clean" is a healthy trajectory. If severity is not dropping, the fixes are not real.

## Rules (non-negotiable)

- Do NOT fix or rewrite the artifact in a review pass. Review it. (Fixing is a separate step.)
- Do NOT invent praise to balance criticism. No "great work, but...".
- Do NOT manufacture filler findings to look thorough. If a pass is genuinely clean, say so and give SHIP IT.
- Tempted to say "looks good"? Look harder first - find the holes, then decide.
- Sweep all five axes before the verdict; do not stop at the first issue.
