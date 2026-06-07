---
name: review
description: Five-axis code review across correctness, readability, architecture, security, and performance. Use when user says "review this", asks for a second opinion before merge, wants you to evaluate code another agent or model wrote, or asks whether a change is safe to ship.
---

# Review

**Approve when the change definitely improves overall code health, even if imperfect. Block when it makes the codebase worse on any axis that matters.** The bar is continuous improvement, not your version of perfect.

When exploring the codebase, use the project's domain glossary so vocabulary in your feedback matches the project's, and check ADRs in the area you're touching. Don't re-litigate decisions that have already been made.

## The five axes

For every change, walk these in order. Stop at the first one that's a problem worth blocking on.

### 1. Correctness

Does it do what it claims?

- Matches the spec / task / ticket the change references.
- Edge cases handled (null, empty, boundary, the inputs that came up in the issue).
- Error paths handled, not just the happy path.
- Tests actually exercise the behaviour, not just the shape of it (data structures, signatures).
- No off-by-one, race condition, or state-leak smells.

### 2. Readability

Could a stranger to this change understand it without the author in the room?

- Names earn their characters. No `data`, `temp`, `result` sitting bare without context.
- Control flow is straight (no nested ternaries, no callback ladders).
- "Clever" is a code smell. Could this be done in fewer lines without losing intent?
- Abstractions earn their complexity. Don't generalise until the third concrete use case.
- Dead code artifacts (`_unused` vars, "// removed" comments, backwards-compat shims for code that's gone) get removed, not left behind.

### 3. Architecture

Does it fit the system?

- Follows existing patterns, or introduces a new one with a stated reason.
- Module boundaries stay clean. Dependencies point the right way.
- No silent duplication that wants to be shared.
- Abstraction level matches the surrounding code (not over-engineered, not too coupled).
- If the change reveals friction worth fixing, note it as a follow-up rather than expanding the change.

### 4. Security

Does the change open new attack surface?

- External input validated at system boundaries before it touches logic or storage.
- No secrets in code, logs, or commit history.
- Auth checks where they belong, including the new endpoint or the new code path.
- DB queries parameterised. Output encoded. No `eval` / `innerHTML` on user data.
- Dependencies introduced from trusted sources, no known CVEs.

### 5. Performance

Does the change introduce a new bottleneck?

- N+1 queries on the hot path.
- Unbounded loops or unpaginated list endpoints.
- Sync work that should be async, or async work that should be batched.
- Large objects allocated in tight loops.
- Cache invalidation paths still correct after the change.

## Severity prefixes

Every comment carries one of these so the author knows what's required vs optional.

| Prefix | Meaning |
|---|---|
| **Critical:** | Blocks merge. Security hole, data loss, broken behaviour. |
| *(no prefix)* | Required before merge. |
| **Optional:** / **Consider:** | Worth thinking about, not required. |
| **Nit:** | Style / formatting. Author may ignore. |
| **FYI:** | Context only, no action. |

Without prefixes, authors treat every comment as mandatory and waste time on nits.

```
Critical: `userId` from the request body goes straight into the SQL string on line 42. Injection vector.
This branch swallows the error from `fetchUser()` instead of surfacing it. Re-throw or return a Result.
Optional: `processOrder` and `processRefund` share 80% of their body. Could share a helper later.
Nit: trailing whitespace on line 17.
FYI: we hit a similar N+1 in the inventory module last quarter; fix is documented in ADR-0011.
```

## Change sizing

Small, focused changes review faster and ship safer.

```
~100 lines  → reviewable in one sitting
~300 lines  → fine if it's one logical change
~1000 lines → split it, unless it's a deletion or mechanical rename
```

If a change mixes refactor + new behaviour, that's two changes. The mix is what hides bugs.

## Process

1. **Understand intent first.** Read the description, linked issue, and tests *before* the implementation. Without intent, a review degenerates into "would I have written it this way".
2. **Read the tests.** Tests reveal what the author thinks the change does. If the tests are weak, that's the first finding. Every other axis is suspect until coverage is honest.
3. **Walk the change with the five axes in mind.** Don't try to hold all five at once. Make one pass per axis if the change is non-trivial.
4. **Categorise findings with severity prefixes.** Group by file and axis so the author can address them as a batch.
5. **Verify the verification.** What did the author run? Tests pass? Build pass? Was anything checked manually? Screenshots for UI? If the verification story is "the tests pass," that's not enough on a non-trivial change.

## Dead code hygiene

After any refactor or replacement, identify orphaned code explicitly. Don't silently delete. List it and ask:

```
DEAD CODE IDENTIFIED:
- formatLegacyDate() in src/utils/date.ts: replaced by formatDate()
- OldTaskCard component: replaced by TaskCard
- LEGACY_API_URL constant: no remaining references
→ Safe to remove these?
```

## Honesty

- **No rubber-stamping.** "LGTM" without evidence of review helps nobody and trains the author to expect lazy reviews.
- **No softening.** "This might be a minor concern" when it's a bug that will hit prod is dishonest. Sycophancy is a failure mode in review.
- **Quantify when you can.** "This N+1 will add ~50ms per row" beats "this could be slow."
- **Push back on broken approaches.** Propose alternatives, don't just complain.
- **Defer gracefully on overrides.** If the author has full context and disagrees, accept it. Comment on the code, not the person.
- **Don't accept "I'll clean it up later."** Later doesn't come. Cleanup happens in this change or as a tracked follow-up with an owner.
