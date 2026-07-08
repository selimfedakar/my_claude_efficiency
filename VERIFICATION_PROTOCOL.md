# VERIFICATION_PROTOCOL.md — Definition of Done & Evidence Discipline

> **Purpose of this document:** The most damaging weak-model behavior is not writing bad code — it is **reporting unverified work as done**. "Fixed ✅" on an unexercised change poisons the owner's trust and the session's state tracking: the owner moves on, the bug resurfaces two sessions later, and now nobody knows which layer it's in. This file defines what "done" means, in levels, and how to report honestly.
>
> **How to use:** Project-independent; referenced by the CLAUDE.md loading paragraph (see THINKING_PROTOCOLS.md) and by FABLE_BAR_RUBRIC.md dimension 1. The verification levels L0–L3 are the shared vocabulary.

---

## The verification ladder

Every claim of completion carries a level. The level must be **stated** in the report.

| Level | Meaning | What it actually proves |
|---|---|---|
| **L0** | Compiles / typechecks / lints | The code is syntactically plausible. Nothing more. |
| **L1** | Test suite passes | Existing invariants hold. Proves the *new* behavior only if a new test targets it. |
| **L2** | **Change exercised end-to-end and its effect observed** | The feature/fix actually works in the running system. |
| **L3** | L2 + adversarial pass + regression sweep | It works AND its neighborhood didn't break AND likely attacks were tried. |

**Minimum bars:**
- Product code change → **L2**. No exceptions for "trivial" changes; trivial changes have a long history of being the ones that break.
- Release-bound work (TestFlight build, deploy, store submission) → **L3**.
- Docs/config-only → L0 plus a read-back of the changed file.

**What L2 concretely means here:** simulator screenshot showing the UI state, a `curl` against the running backend with the response pasted, a driven flow in the app (tap → observe), a WebSocket message sent and the action observed. *"The code should now do X"* is L0 wearing an L2 costume.

## The evidence table

Any report claiming completion of substantial work includes this table. One row per claim. If a row's evidence cell would be empty, the claim moves to "Assumed" or "Unknown" — it does not stay in the table with hand-waving.

```md
| Claim | Level | Evidence |
|---|---|---|
| /reserve/count returns live count | L2 | curl https://…/reserve/count → 200 {"count":0} |
| Rate limit rejects 6th request   | L2 | 6 rapid curls → 5×200 then 429, body pasted below |
| Existing suite unaffected        | L1 | pytest → 363 passed in 41.2s (output pasted) |
| Old clients tolerate new field   | assumed | new field is additive; NOT tested against old build |
```

Rules:
- **Numbers are pasted, never recalled.** "363 tests pass" is only writable next to actual pytest output from this session. Reporting a remembered count as a fresh result is fabrication.
- Evidence is the *observation*, not the intention: "ran curl, got 200 + body" — not "endpoint was implemented correctly".

## The regression sweep (protects the neighborhood)

A change is not done when *it* works; it's done when *nothing else stopped working*. Weak models verify the target and ignore the blast radius.

**Procedure:**
1. For every function/component/endpoint modified: `grep` its call sites (T1 already produced this list — reuse it).
2. For each call site, one line: unaffected because X / affected and verified how.
3. Shared modules get double weight: anything under a `shared/` path is consumed by both sides — verify **both** sides (e.g. backend tests AND mobile typecheck+behavior), not just the side you were working on.
4. If the diff touched any screen with keyboard/composer/bottom-anchored UI, re-drive that screen — this class regresses silently (see the relevant P-entries).

## The adversarial pass (L3)

Before declaring release-bound work done, switch stance from builder to attacker:

1. Write **3 realistic scenarios that would break this change.** Realistic = things this system's users/network/lifecycle actually do: double-tap, backgrounding mid-stream, socket drop with requeue, old client version, permission denied. (Pull from the T7 grid.)
2. Actually run the top scenario. Reason carefully through the other two if running them is impractical — and label which was which.
3. If you cannot think of 3, you don't understand the change's environment yet — that itself is a finding. Go read more (T1).

## Honest reporting language

The report's words must encode certainty precisely. This is not style — the owner makes go/no-go decisions on these words.

**Three buckets, always separated:**
- **Verified:** stated plainly, with evidence. "Rate limiting works: 6 rapid requests → 5×200, 1×429."
- **Assumed:** labeled. "Assumed: Railway env vars unchanged since last deploy — not re-checked."
- **Unknown / not done:** listed explicitly. "Not verified on physical device — simulator only. BLE path untouched and untested."

**Banned phrases** — each must be replaced by evidence or by a labeled unknown:
- "should work" → run it, or write "unknown: not exercised"
- "probably fine" → verify, or name the exact risk left open
- "seems to" / "I believe" (about your own change) → observe it
- "everything passes" → paste the counts
- "done ✅" on anything below its minimum bar

**Failure is reported as failure.** If the fix didn't work, the report says "not fixed — attempted X, symptom persists, current best hypothesis Y". A weak model's instinct is to present the *most optimistic* reading of ambiguous results; the rule is to present the *most accurate* one. Bad news early is cheap; bad news late is expensive.

## Session-state honesty

At session end, the state snapshot (CLAUDE.md "State as of" + session report) records verification levels, not vibes:

```md
- WS instant-action channel: L2 (sim-verified, POST→Maps <2s observed)
- Requeue-on-reconnect: L1 only (unit-tested; NOT driven with real socket drop)
- Physical device path: L0 — untested, hardware pending
```

This is what makes the next session (possibly a weaker model, possibly months later) able to trust the file — it inherits *calibrated* state, not optimism.

---

## Quick card (the whole file in six lines)

1. Every "done" carries a level: L0 compiled · L1 tests · L2 exercised+observed · L3 +adversarial+sweep.
2. Product code = L2 minimum; release-bound = L3.
3. Evidence table: claim | level | pasted observation. Numbers are pasted, never recalled.
4. Sweep the call sites; shared code = verify both consumers.
5. Three buckets in every report: Verified / Assumed / Unknown. Banned: "should work".
6. Failure reported as failure, with best hypothesis — never dressed as success.

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` set, layer 2: cognition transfer.*
