# FABLE_BAR_RUBRIC.md — Self-Review Rubric: Does This Output Meet the Bar?

> **Purpose of this document:** This is the tester file. Before presenting any substantial output (code change, plan, review, debugging conclusion, report), the model **switches roles** — from builder to skeptical senior reviewer — and grades its own work against the rubric below. Work that fails the bar gets fixed *before* the owner sees it, not after.
>
> This rubric was written by a stronger model grading the typical failure modes of weaker ones. That is its value: the dimensions below are precisely where outputs degrade when raw capability drops, so checking them is where the gap closes.
>
> **How to use:** Referenced from every project's CLAUDE.md loading paragraph. Procedure at the bottom. Dimensions cite the protocols (T1–T8) in THINKING_PROTOCOLS.md and levels (L0–L3) in VERIFICATION_PROTOCOL.md.

---

## The eight dimensions

Score each 1–5. The anchors describe what a 2 and a 5 actually look like — score against the anchors, not against feelings.

### D1 — Correctness evidence *(weight: highest)*
Was the change proven to work, at the right verification level?
- **2:** Compiles/typechecks; "should work now"; tests maybe run, counts recalled from memory.
- **5:** L2+ with pasted observations; evidence table present; new behavior has a targeted test; failure paths tried, not just the demo path.

### D2 — Root-cause depth
Does the work address the mechanism or the symptom?
- **2:** Null check added where the crash was; retry wrapped around the flaky call; no answer to "why did this state occur?".
- **5:** Causal chain explained in one paragraph; fix placed at the deliberately chosen level (T3), with the choice stated; symptom-level fixes carry a recorded debt note.

### D3 — Scope integrity
Was the blast radius handled — and nothing beyond it touched?
- **2:** Target file edited in isolation; callers unchecked; OR the diff "helpfully" refactored five unrelated things nobody asked for.
- **5:** Call sites enumerated and dispositioned (T1); shared modules verified on both consumer sides; diff contains exactly the asked-for change; out-of-scope discoveries reported as notes, not smuggled edits.

### D4 — Simplicity / least mechanism
Is this the smallest solution that genuinely works?
- **2:** New abstraction layer, new dependency, or new "manager/service/helper" file for a problem the codebase already solves; config options nobody requested; diff 10× the size of the ask.
- **5:** T5 order visibly applied — deletion or reuse chosen when possible; new abstraction only with 3 existing usages; net diff proportional to the problem.

### D5 — Edge coverage
Were the standard failure surfaces swept?
- **2:** Happy path only; empty list, first run, offline, double-tap, old-client cases never mentioned.
- **5:** T7 grid walked; each relevant cell handled or dismissed with a stated reason; the interesting cells (requeue duplicate, mid-operation interruption) actually exercised for real-time code.

### D6 — Codebase & knowledge fit
Does the work behave like it was written by the person who wrote the rest of the repo?
- **2:** New idiom parallel to an existing one; reinvented util; ignores PATTERNS.md and repeats a documented dead-end; proposes a DECISIONS.md-rejected alternative.
- **5:** Matches file's existing style/error-handling/naming; searched for prior art first (T4.3); cites P-entries and D-entries when relevant ("per P-007, composer uses KAV + header height").

### D7 — Honest reporting
Do the report's words match the actual epistemic state?
- **2:** "Done ✅, everything works"; optimistic reading of ambiguous results; failures buried mid-paragraph; banned phrases ("should work") present.
- **5:** Verified / Assumed / Unknown cleanly separated; failures led with, not hidden; numbers pasted; the owner could make a go/no-go decision from this report without asking a single follow-up.

### D8 — Rules compliance
Were the owner's standing laws honored, end to end?
- **2:** A git commit ran "to be helpful"; multi-file commit proposed; Turkish comment in source; AI attribution in a message; wrong Metro port; session ended without its report.
- **5:** T8 refresh done before delivery; commit plan is one-file-per-commit for the owner to run; English-only sources; no attribution; ports respected; session ritual complete.

## Pass bar

- **No dimension below 3, and average ≥ 4 → present the work.**
- Any dimension at 1–2 → **fix it first**, then re-grade. Maximum two self-iterations; if still failing, present anyway but **lead with the gaps** ("D5 at 2: offline path unhandled, needs a decision") — an honestly flagged gap passes; a hidden one never does.
- Report the grading result in one line with the delivery: `Self-review: D1:5 D2:4 D3:5 D4:4 D5:3 D6:5 D7:5 D8:5 — pass. Weakest: D5 (offline case dismissed, reason stated).`
- Do not inflate. This rubric only works if a 2 is called a 2. Grading yourself 5s by default converts the tool into decoration — and the owner will recalibrate by hand the first time an inflated 5 fails in production, destroying the rubric's value permanently.

## The smell list — instant tells of below-bar output

If any of these appear in your own draft, the relevant dimension is at 2 regardless of how the prose feels. Sorted by dimension:

- **D1:** "should work", "probably", a test count not pasted from this session's run, no observation of the running system.
- **D2:** a null/undefined guard with no sentence explaining the null's origin; `try/catch` swallowing broadly "for safety"; a retry added to a deterministic failure.
- **D3:** an edited function whose callers were never greped; a "while I was there" refactor mixed into a bugfix diff.
- **D4:** a new file whose name contains `Manager`, `Helper`, `Utils`, or `Service` duplicating existing capability; a new dependency for <30 lines of logic; an interface with one implementation "for testability" nobody asked for.
- **D5:** zero mention of the empty/first-run/offline cases in feature work; real-time code with no reconnect/duplicate story.
- **D6:** code that solves a problem PATTERNS.md already solved differently; a proposal matching a rejected D-entry with no mention of its "Revisit if" condition.
- **D7:** the word "done" above L2; good news first when the news is mixed; ambiguity resolved in the optimistic direction.
- **D8:** any `git commit` in the transcript; a commit message touching two files; a session about to end with no report written.

## Worked micro-example — same task, below vs at the bar

**Task:** "Users report the waitlist count sometimes shows 0 even though people signed up."

**Below the bar (fails D1, D2, D5, D7):**
> Fixed! The count was probably a caching issue, so I added `cache: "no-store"` to the fetch and a fallback so it never shows 0 — if the count is 0 it now shows nothing. Should work now. ✅

*Why it fails: no reproduction, no root cause ("probably"), the fallback **hides the symptom instead of fixing it** (a real count of 0 for a new product is correct data — see D2 anchor), nothing verified, edge case "legitimately zero" actively broken while "fixing" it.*

**At the bar:**
> Reproduced, root-caused, fixed at the right level, L2-verified.
> **Cause (T2 — 3 hypotheses tested):** not client caching (hard-refresh reproduced it), not the DB (direct query returned 4). Discriminating test: `curl` on the endpoint returned 0 with an edge-cache header — the endpoint's CDN cache TTL predates the signups. Data was stale at the edge, not wrong at origin.
> **Fix (T3 — at cause, not symptom):** count endpoint now sets `Cache-Control: s-maxage=60` (was 86400). Client untouched — a "hide zero" fallback was considered and rejected: zero is a legitimate state pre-launch and the badge already has copy for it.
> **Verified:** curl → 200 `{"count":4}`, response header shows new TTL; badge in sim shows "4 people ahead of you" (screenshot). Suite: 363 passed (output below).
> **Assumed:** 60s staleness acceptable for a marketing badge — flag if you want it tighter.
> Self-review: D1:5 D2:5 D3:5 D4:5 D5:4 D6:5 D7:5 D8:5 — pass.

The difference is not writing quality. It is that every rubric dimension forced an action the first version skipped.

## Self-review procedure (the loop)

1. Finish the draft (code + report).
2. **Switch roles.** You are now a senior reviewer who did not write this and is mildly suspicious of whoever did. Re-read the *diff and report*, not your memory of them.
3. Scan the smell list. Any hit → dimension is a 2.
4. Score D1–D8 against the anchors.
5. Below bar → fix the failing dimensions (this usually means going back to a T-protocol you skipped, not polishing prose). Re-grade once.
6. Present with the one-line score, weakest dimension named.

Cost: 3–5 minutes. It is the last chance to catch a below-bar output while it's still free to fix.

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` set, layer 2: cognition transfer. This rubric encodes what a stronger model checks implicitly before speaking; run it explicitly and the output converges toward that bar.*
