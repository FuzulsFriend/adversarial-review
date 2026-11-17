# A sample review (SYNTHETIC)

The artifact below is a made-up spec with planted flaws. The review is what the skill produces for it.

## The artifact

```text
SPEC: Password reset by email
1. A user requests a reset; we email a link containing a token. The token is valid for 24 hours.
2. Clicking the link opens the reset form. The user sets a new password and is logged in.
3. After a successful reset, all other sessions are signed out.
4. Tokens are stored in the `reset_tokens` table (user_id, token, created_at).
5. We rate-limit reset requests to protect the mail provider.
(Signup allows phone-only accounts with no email address.)
```

## The review

### Findings

- **Severity:** Blocker
- **What:** Reset tokens are stored in plain text, so a read-only database leak is an account takeover.
- **Where:** Step 4 (`reset_tokens.token`).
- **Proof:** Step 4 lists the stored column as `token`, the same value that step 1 puts in the emailed link. Nothing says it is hashed, and a link token that is verified by lookup has to be stored as sent.
- **Why it matters:** Anyone with read access to a backup or a replica can reset any account that has a pending token, with no email access.
- **Fix:** Store a hash (SHA-256 of a 32-byte random token), compare hashes in constant time, and say so in the spec.

- **Severity:** Blocker
- **What:** The flow cannot work for phone-only accounts, and the spec does not say what happens to them.
- **Where:** Step 1 versus the note after step 5.
- **Proof:** Step 1 sends the token by email. The note says signup allows accounts with no email address.
- **Why it matters:** Those users get no reset path, or the implementer invents one (SMS, support ticket) with a different security level.
- **Fix:** Decide: phone-only accounts reset by SMS code with its own expiry and rate limit, or they cannot reset and the spec says so.

- **Severity:** Major
- **What:** The token is not single-use, and nothing invalidates older tokens when a new one is requested.
- **Where:** Steps 1, 2 and 4.
- **Proof:** Step 4's table has no `used_at` or deletion rule, and step 2 never consumes the token.
- **Why it matters:** A forwarded or leaked email stays a working password reset for 24 hours, even after the user has already reset.
- **Fix:** Mark the token used on success, delete all tokens for the user, and invalidate earlier tokens on each new request.

- **Severity:** Major
- **What:** Logging the user in straight after a reset turns an emailed link into a login credential.
- **Where:** Step 2.
- **Proof:** Step 2 says "and is logged in" with no second factor check.
- **Why it matters:** Accounts with two-factor authentication lose it for anyone holding the email.
- **Fix:** Redirect to the login page, or require the second factor before creating the session.

- **Severity:** Minor
- **What:** "Rate-limit" has no numbers or scope.
- **Where:** Step 5.
- **Proof:** Step 5 names no limit, window or key (per user, per IP, per address).
- **Why it matters:** A builder can implement a limit that protects the mail provider but still lets an attacker flood one user's inbox.
- **Fix:** For example 3 requests per hour per account and 20 per hour per IP, with the same response whether or not the account exists.

### Hard Questions

1. Does the reset response reveal whether an account exists? If it does, this is a user-enumeration endpoint.
2. Who is allowed to trigger a reset for an account with an active paid subscription, and is an email-only recovery acceptable for it?
3. What is the support path when the user has lost the mailbox as well?

### Verdict

**NOT READY** - 2 Blockers and 2 Majors, and 3 open hard questions. Fixing the token storage and the phone-only case will change the table and the flow, so re-review after the fix and look for what the fix broke.
