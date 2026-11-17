# Triage: a findings list is not a to-do list (SYNTHETIC)

A small PR (5 files) came back from review with six findings. The skill's rule: sort every finding into fix-now, fold-in, or defer-and-document, and name why each deferral is acceptable. Fixing all six would over-engineer a 5-file change.

| # | Severity | Finding | Decision | Why |
|---|---|---|---|---|
| 1 | Blocker | A race double-sends emails | **fix-now** | Real, user-visible, cheap to close with a unique key |
| 2 | Major | User input is rendered into HTML unescaped | **fix-now** | Security; one-line escaping at the template |
| 3 | Medium | Persisted error strings may contain personal data | **fold-in** | Same file as #1; strip the field while there |
| 5 | Low | The run-history table grows without bound | **fold-in** | Add a retention line to the job already being touched |
| 4 | Medium | Build a full sending-state machine and idempotency-key system to close a rare hard-kill window | **defer, documented** | Risk: a duplicate email after a hard process kill, observed 0 times; cost: a new subsystem larger than the PR. Revisit if it is ever observed |
| 6 | Low | Replace the static bearer auth with a per-request signature scheme | **defer, documented** | The endpoint is internal and rate-limited; the scheme is a separate piece of work with its own review |

After the fixes, the review runs again and hunts specifically for what the fixes broke (a new unique key can reject a legitimate retry). Trajectory to aim for: pass 1 has a Blocker and a Major, pass 2 has one Minor, pass 3 is clean.
