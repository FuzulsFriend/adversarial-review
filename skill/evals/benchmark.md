# adversarial-review - eval iteration 1

Baseline = same prompt, no skill. With-skill = skill read + applied. 2 artifacts, each with planted flaws.

## Eval 0 - flawed mini-spec (planted: UTC/local contradiction [blocker], webhook idempotency [major], ambiguous "promptly" [minor], phone-only-vs-email [blocker/hard-q])

| Assertion | with_skill | baseline |
|---|---|---|
| Caught UTC-vs-local contradiction | PASS | PASS |
| Caught missing webhook idempotency | PASS | PASS |
| Caught ambiguous "promptly" | PASS | PASS |
| Caught phone-only-vs-email | PASS | PASS |
| Severity-ranked findings (Blocker/Major/Minor) | PASS | FAIL (prose) |
| Explicit Hard Questions section | PASS (6) | FAIL (folded into prose) |
| Binary verdict (NOT READY) | PASS | PASS ("Not ready") |
| No reassurance padding | PASS | FAIL ("What's fine" section) |
| Bonus real findings beyond planted | signature verification, wrong Stripe event name, unbounded reminder loop | signature verification, wrong Stripe event name |
| Planted-flaw recall | 4/4 | 4/4 |

## Eval 1 - flawed plan step (planted: use-after-delete [blocker], scope-balloon migration [blocker], blunt cache flush [major], missing acceptance/error handling)

| Assertion | with_skill | baseline |
|---|---|---|
| Caught use-after-delete ordering | PASS | PASS |
| Caught scope-balloon legacy migration | PASS | PASS |
| Flagged the blunt cache.invalidateAll() | PASS | PASS |
| Flagged missing error handling / acceptance | PASS | PASS |
| Severity-ranked findings | PASS | FAIL (prose) |
| Explicit Hard Questions section | PASS (5) | FAIL |
| Binary verdict | PASS (NOT READY) | PASS ("do not proceed") |
| Bonus real findings beyond planted | Windows /tmp path bug, concurrent-run temp-file collision | none beyond planted |
| Planted-flaw recall | 4/4 | 3.5/4 |

## Analysis

- **Flaw recall** was high in BOTH arms - these planted flaws are fairly obvious and the model is strong, so the skill did not dramatically raise raw catch rate on this easy set.
- **Where the skill clearly wins:** (1) structured, severity-ranked, scannable output with explicit Where/Why/Fix; (2) an explicit Hard Questions section (baseline buried these in prose); (3) an enforced binary verdict; (4) NO reassurance padding (baseline added "What's fine"); (5) more depth - the with-skill runs surfaced extra REAL bugs the baseline missed, notably the Windows `/tmp` path bug and the concurrent-temp-file collision in eval 1.
- **Bug found in the skill:** the output-format spec produced a doubled severity line ("[Blocker]" then "Major"). Fixed in this iteration (severity is now a single `**Severity:**` field).
- **Honest caveat:** this is a 2-artifact, planted-flaw test on an easy set. The skill's value is most visible on output discipline + depth here; the gap on raw recall would likely widen on subtler artifacts. A larger/subtler test set would measure that.
