# REVIEW-CHECKLIST.md — Frozen Review Standards

> **Purpose of this document:** Guide + master checklist for a `docs/REVIEW-CHECKLIST.md` in every project. A strong model reviews code by *taste* — it notices what matters without being told. A weaker model reviews by *list* — it is excellent at verifying explicit criteria and mediocre at noticing unlisted ones. This file freezes the strong model's taste into an explicit list, so review quality survives the model swap.
>
> **How to use:** Copy the master checklist below into each project's `docs/REVIEW-CHECKLIST.md`, delete sections that don't apply, and add the project-specific section from CLAUDE.md landmines + PATTERNS entries. Then instruct any model: *"Review this diff against docs/REVIEW-CHECKLIST.md; report findings per item, most severe first; say which items you checked and found clean."* — the "found clean" requirement forces the checklist to actually be walked, not skimmed.

---

## Design principles for the checklist itself

1. **Binary over vague.** Every item must be checkable yes/no against a diff. "Is the code clean?" is not an item. "Does any function swallow an exception without logging or rethrowing?" is an item.
2. **Ordered by severity.** Correctness → security → data safety → concurrency → platform rules → performance → style. A weak model with limited attention spends it top-down; put the expensive mistakes first.
3. **Project section outranks generic section.** The generic list catches textbook bugs; the project list catches *your* bugs. Feed it from PATTERNS.md: every anti-pattern with a `P-NNN` becomes a one-line review item referencing it.
4. **Keep it under ~60 items.** Beyond that, walk-through discipline collapses. Prune items that have never fired.

---

## Master checklist (copy into projects, prune, extend)

### A. Correctness (highest severity)

- [ ] A1. Off-by-one: every loop bound, slice, and pagination cursor — is the boundary element included/excluded intentionally? (`<` vs `<=`, `len` vs `len-1`)
- [ ] A2. Null/undefined/None flows: can any newly-introduced value be null on a path where it's dereferenced? Especially: optional API fields, first-run states, empty lists.
- [ ] A3. Error paths: does every `catch`/`except` either handle meaningfully, log with context, or rethrow? Flag ANY silent swallow.
- [ ] A4. Async correctness: every promise awaited or explicitly detached with a comment; no floating promises; no `await` inside loops that should be parallel (or vice versa — parallel writes that must be sequential).
- [ ] A5. State machine integrity: if the project has an explicit state machine (e.g. a connection state machine), does the change route ALL state transitions through it? Flag any new ad-hoc boolean that shadows machine state.
- [ ] A6. Time and timezone: any date math using local time where UTC is required, or naive datetime mixing? Absolute timestamps stored, local only at display?
- [ ] A7. Idempotency of retried operations: if this code can run twice (retry, reconnect, requeue, webhook redelivery), is running it twice safe? This is a standing trap in WS/requeue designs.

### B. Security & secrets

- [ ] B1. No secret, API key, token, or credential in code, logs, error messages, or committed config. Check diff for high-entropy strings.
- [ ] B2. All user/device input validated server-side; length limits on anything persisted; no string-built SQL/shell.
- [ ] B3. Auth is fail-closed: if the auth check errors, the request is REJECTED (401/403), not allowed through. (Verified pattern in this stack — keep it that way.)
- [ ] B4. New endpoints: rate-limited if public-facing? Deliberately public endpoints (e.g. a public count/reserve route) documented as such?
- [ ] B5. PII: nothing sensitive in analytics events, crash reports, or logs.

### C. Data safety & migrations

- [ ] C1. Schema/data-shape changes backward compatible with data already in production? (Live-app rule: existing user data is sacred.)
- [ ] C2. Destructive operations (DELETE, overwrite, drop) gated behind explicit confirmation or unreachable from normal flows?
- [ ] C3. Client and server tolerate each other's OLD versions — users do not update in lockstep with deploys.

### D. Concurrency & real-time

- [ ] D1. Shared mutable state accessed from multiple tasks/threads/callbacks: protected or redesigned?
- [ ] D2. WebSocket/stream lifecycle: reconnect handled; buffers bounded; no listener leaks on unmount/close; requeue does not duplicate (see A7).
- [ ] D3. Races between rapid user actions (double-tap submit, navigate-during-fetch): is there a guard?

### E. Platform & release gates (mobile-heavy projects)

- [ ] E1. New native capability (mic, calendar, contacts, local network, Bluetooth) → Info.plist/permission string present, honest, and review-safe? Cross-check every past store rejection recorded in PATTERNS.md — is its fix still intact in this diff?
- [ ] E2. Permission requests: requested at the moment of use with rationale, never "request-and-alert" at launch (known review risk).
- [ ] E3. iOS keyboard/safe-area/navigation quirks: does the diff touch any screen with a composer or bottom-anchored UI? Verify against the established keyboard pattern (KAV + header height — see the relevant P-entry).
- [ ] E4. Background behavior: does the change assume the app is foregrounded? What happens on AppState change mid-operation?

### F. Performance (flag only with evidence)

- [ ] F1. N+1: new loop containing an await/query/render that should be batched.
- [ ] F2. Unbounded growth: any list/cache/log that only ever appends.
- [ ] F3. Re-render storms: new context values / inline objects passed deep into React trees; missing memoization ONLY where a measurable hot path exists (do not cargo-cult memo).
- [ ] F4. Payloads: are we shipping fields the client never reads?

### G. Tests & verification

- [ ] G1. New behavior has a test; the FIX for any bug has a regression test reproducing the original symptom.
- [ ] G2. Tests assert behavior, not implementation (would survive a refactor).
- [ ] G3. Full suite green — stated with the actual count, not "should pass".
- [ ] G4. Was the change *exercised* end-to-end (simulator/curl/browser), not just compiled? State what was driven and observed.

### H. Consistency & maintainability (lowest severity — report, don't block)

- [ ] H1. Change follows the file's existing idiom (naming, error style, comment density) — no parallel second way of doing something the codebase already does.
- [ ] H2. No new dependency when the codebase already contains the capability.
- [ ] H3. Dead code, commented-out blocks, stray debug logs removed.
- [ ] H4. English-only in source files, comments, and docs (owner rule).
- [ ] H5. Diff is atomically committable — one concern per file-change, compatible with the one-commit-per-file rule.

### Z. Project-specific items (HIGHEST priority — populate per project)

> Generate this section from CLAUDE.md's Landmines + PATTERNS.md entries.
> Every entry that can be violated by a diff becomes one line here. Examples:

- [ ] Z1. No git commit/push executed by the model — plans only, owner runs them.
- [ ] Z2. No AI/Claude reference in any commit message, comment, or user-facing string.
- [ ] Z3. Metro/dev-server ports respect the cross-project port map (each project has its assigned port — check CLAUDE.md).
- [ ] Z4. Shared modules (e.g. a shared endpointing policy) changed in ONE place only; no forked copies.
- [ ] Z5. Brand assets follow recorded design decisions (e.g. gradient wordmark stays gradient — never flattened).

---

## How to run a review with this file (instructions for any model)

1. Read `docs/REVIEW-CHECKLIST.md` fully before reading the diff.
2. Read the diff; for each checklist item, actively hunt for violations — do not passively wait for them to jump out.
3. Report findings ordered by section severity (A first). Each finding: item ID, file:line, one-sentence defect, concrete failure scenario (inputs → wrong outcome).
4. Explicitly list the sections checked and found clean — this proves coverage.
5. Do not report style nits from section H as blockers; label them "non-blocking".
6. If a finding contradicts a D-entry or P-entry, cite it (`violates P-007`) — that citation is what earns the finding instant trust.

## Maintenance

- Every bug that escapes to production or to the owner's manual testing = a missing checklist item. Add it within the same session, in the right severity section, referencing its P-entry.
- Once a quarter, prune: any item that has never produced a finding in ~10 reviews is either always-followed (keep if cheap) or irrelevant (delete).

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` knowledge-distillation set.*
