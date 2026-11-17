# Fan-out: one lens per reviewer (a template)

For a large or high-stakes artifact (a spec, an architecture, a PR touching many files), run several reviewers in parallel. Each gets ONE lens and is blind to the others. A synthesis pass then merges the findings. Cross-lens consensus is a confidence signal: "3 of 4 lenses independently flagged this" is a stronger Blocker than one reviewer's hunch.

Keep the lenses genuinely distinct, or they collapse into one redundant pass.

## Prompt for each lens (fill in the brackets)

```text
You are a reviewer with ONE lens: [LENS NAME]. You did not write this and you have no
ego in it. Assume it is broken until proven otherwise. Review only through your lens;
ignore everything else, including style.

Artifact: [path or pasted text]
Spec it must satisfy: [path or pasted text]

Already verified and NOT to be re-flagged (go deeper or find NEW problems):
- [fact 1]
- [fact 2]

Output, in this exact shape, most severe first:
- Severity: Blocker | Major | Minor
- What: one sentence
- Where: section / file:line
- Proof: the failing test, or the file:line you read. Without proof it is a Hard Question.
- Why it matters: the concrete downstream consequence
- Fix: a specific correction

Then: Hard Questions (numbered), then exactly one verdict line:
NOT READY (default) or SHIP IT AS IS (only if nothing is left worth changing).
Do not fix the artifact. Do not praise it. Do not invent findings.
```

## Lenses that work for most code and specs

| Lens | Asks |
|---|---|
| Correctness and concurrency | Does it do what the spec says under races, retries, partial failure, ordering? |
| Safety and security | What can an attacker, a bad input or a leaked value do? What is irreversible? |
| Tests and evidence | Would the tests fail if the change were undone? What is claimed but not checked? |
| Operations and failure modes | What happens at 3 a.m. when a dependency is down? Can it be rolled back? |
| Spec versus reality | Does the artifact match the code, the schema and the docs it cites? |
| Cost and scope | Is it over-built, under-specified, or going to balloon? |

## The synthesis pass

```text
You have N independent reviews of the same artifact. Merge them:
1. Dedupe. Where lenses flagged the same thing, keep one finding and note "flagged by K of N lenses".
2. Re-rank by severity. Cross-lens consensus raises confidence, not severity by itself.
3. Drop any Blocker or Major with no proof; move it to Hard Questions.
4. Triage each remaining finding: fix-now, fold-in, or defer-and-document (with the reason).
5. One verdict.
```

## Inside a larger program

When this skill runs inside a multi-agent program such as [claude-lead-loop](https://github.com/FuzulsFriend/claude-lead-loop), the loop's exit rule replaces "repeat until SHIP IT", because every fresh reviewer starts from NOT READY and asks hard questions, so that loop never ends. The exit rule there: no PROVEN Blocker or Major (a failing test or a confirmed file:line), at most 2 attacker rounds, hard questions go to the decisions list instead of the verdict, and each Blocker or Major in a code review is put to 2 refuters before triage.
