# THINKING_PROTOCOLS.md — How a Strong Model Thinks, Written as Procedure

> **Purpose of this document:** The six project files (CLAUDE.md, DECISIONS, PATTERNS, skills, checklist, roadmap) transfer *knowledge*. This file transfers *cognition*. A strong model does the things below implicitly, without being asked. A weaker model does them reliably **only when they are written as mandatory procedure**. This file is that procedure.
>
> **How to use:** This file is project-independent. Reference it from every project's CLAUDE.md with the loading paragraph at the bottom. The protocols are numbered T1–T8; other files (rubric, verification) refer to them by ID.
>
> **The core insight:** Most of the quality gap between a stronger and weaker model is not raw intelligence — it is *discipline under uncertainty*: the strong model refuses to act before understanding, holds multiple hypotheses, and distrusts its own first answer. Every protocol below converts one of those habits into an explicit, checkable step.

---

## T1 — Read before write (the anti-pattern-matching rule)

Weaker models edit code by *pattern recognition*: the code looks like a familiar shape, so they edit the shape they remember, not the code that exists. This produces edits that are plausible and wrong.

**Procedure — before editing any existing code:**
1. Read the function you are changing AND the functions it calls that you rely on. Not skim — read.
2. Find every call site of what you're changing (`grep` the symbol). List them. For each: does my change break its expectations?
3. State (to yourself, in one line) the invariant the current code maintains that your edit might violate. If you cannot name one, you have not read enough.
4. **The quoting test:** if you cannot quote the actual lines your change depends on, you don't know them — you're guessing. Go read them.

**Banned:** editing a file you have not opened this session; "fixing" a function signature without checking callers; assuming a helper does what its name says.

## T2 — Differential debugging (the anti-first-idea rule)

The single biggest weak-model failure: grabbing the first plausible cause and fixing it. The first plausible cause is wrong often enough that acting on it unverified is negative-value work — it adds a wrong fix on top of the original bug.

**Procedure — when anything behaves unexpectedly:**
1. Write the symptom precisely (exact error, exact wrong value, exact conditions). Vague symptom = vague debugging.
2. Generate **minimum 3 hypotheses at different layers** before touching anything:
   - data layer (the input/state is not what I think)
   - logic layer (the code does not do what I think)
   - environment layer (config, version, port, cache, simulator-vs-device, deployed-vs-local)
   Timing/concurrency is a fourth layer for anything real-time.
3. Rank by prior probability, then run the **cheapest discriminating experiment** — the observation that splits the hypothesis space, usually a log line, a curl, or reading one more file. Not a fix. An observation.
4. Only write the fix after the evidence names the cause. **A fix you cannot explain is not a fix** — even if the symptom disappears, you've hidden the bug, not killed it.
5. **Two failed fixes = full stop.** The bug is now by definition a 2+ iteration bug: stop fixing, re-open the hypothesis space, and explicitly question one assumption from your T6 ledger — the wrongness is usually there, not in the code you keep staring at. (And it now owes a PATTERNS.md entry.)

**First move for any bug:** search PATTERNS.md for the symptom. Paid-for knowledge beats fresh reasoning.

## T3 — Root-cause gate (the anti-symptom-patch rule)

Weak-model tell: the null-check patch. Something was null, so a null check appears — and the question *"why was it null?"* is never asked. Ten sessions later the codebase is defensive spaghetti and the real bug still lives.

**Procedure — after finding the proximate cause:**
1. Ask "why was the system able to reach this state?" up to three times. Stop when you hit design intent or external input.
2. Decide **deliberately**: fix at symptom, proximate cause, or root — and say which you chose and why. Fixing at the symptom is sometimes right (release pressure, root is out of scope) but it must be a *stated decision*, not a default.
3. If you fix at the symptom, record the debt: one line in the session report or a ROADMAP deferral, so the root doesn't vanish.

## T4 — Altitude check (the anti-literalism rule)

Weaker models solve the *literal* request at the *literal* location. Strong models first ask whether the request itself is right and whether the fix belongs one level up.

**Procedure — before implementing any non-trivial request:**
1. **Right problem?** Does the request make sense given ROADMAP.md and DECISIONS.md? If it conflicts (rebuilds a rejected alternative, pre-builds a deferred item), say so BEFORE implementing — that flag is worth more than the implementation.
2. **Right level?** If this fix is needed here, is the same bug waiting at sibling call sites? Should the fix live in the caller, the shared module, or the design instead of this leaf?
3. **Already exists?** Search the codebase for an existing mechanism before building one. The strongest single predictor of weak output is *the parallel second implementation* of something the repo already does.

Timebox: this is 2–3 minutes of thought, not a design review. Its output is either "proceed as asked" or one short paragraph to the owner.

## T5 — Least mechanism (the anti-complexity rule)

Weak models add; strong models subtract. Given a problem, consider solutions **in this strict order**, taking the first that genuinely works:

1. **Delete code** — can removing something make the problem impossible?
2. **Use existing code** — configuration, reuse, or a small extension of what's there.
3. **Smallest local change** — a few lines in place.
4. **New abstraction** — allowed ONLY with 3 concrete usages existing *today*. "We'll need it later" is not a usage; ROADMAP's *Later* bucket explicitly forbids pre-building for it.

Corollary: net negative line count is a feature. If your diff for a small ask is large, that is a signal to stop and re-derive, not to keep going.

## T6 — Assumption ledger (the anti-house-of-cards rule)

Strong models track what they *know* vs what they *assume*; weak models promote assumptions to facts silently, then build hours of work on one wrong one.

**Procedure — at the start of any multi-step task:**
1. List the assumptions the plan rests on, each tagged `[VERIFIED — how]` or `[UNVERIFIED]`.
   *Example: "backend deployed version includes the /reserve/count route [VERIFIED — curl returned 200]" vs "Metro port 8082 is free [UNVERIFIED]".*
2. Any UNVERIFIED assumption that would **invalidate the whole approach** if false must be checked before the main work begins. Cheap check now beats rework later.
3. When debugging stalls (T2 step 5), this ledger is the suspect list.

## T7 — Edge-case grid (the anti-happy-path rule)

Weak models test the demo path. Strong models mentally sweep a standard grid. For any new behavior, walk this list and handle or consciously dismiss each:

- **Cardinality:** empty / exactly one / many / maximum realistic.
- **Lifecycle:** first run ever (no stored state) / after update from an older version / after reinstall.
- **Connectivity:** offline at start / connection drops mid-operation / reconnect with a queued duplicate (requeue safety).
- **Interruption:** app backgrounded mid-flow / user double-taps submit / navigates away during fetch.
- **Compatibility:** old client against new server, and the reverse — users don't update in lockstep with deploys.
- **Input:** very long, unicode/emoji, whitespace-only, hostile.
- **Permission:** denied / revoked while running / "ask every time".

"Consciously dismiss" means one clause: "concurrent duplicate impossible here because X". The grid takes minutes; the production bugs it prevents take sessions.

## T8 — Constraint refresh (the anti-drift rule)

Weak models forget standing rules as the session grows — the rules scroll out of attention, and a git commit gets run in session-hour three that never would have been run in minute five.

**Procedure:**
1. Before any **commit plan, report, deploy, or file-set delivery**, re-read the Workflow Rules section of the project's CLAUDE.md and check the output against it item by item.
2. On long sessions, re-read it whenever you notice you're operating from memory of the rules rather than the rules.
3. Highest-risk drift rules in this workspace, hardcoded here as a backstop: **no git commit/push ever (owner runs them); atomic one-file commits; English-only source and docs; no AI attribution anywhere; per-project port map.**

---

## Loading paragraph (paste into every project's CLAUDE.md)

```md
## Thinking protocols (mandatory)
Read ~/claude_efficiency/THINKING_PROTOCOLS.md at session start. Non-negotiable triggers:
any bug → T2 (3 hypotheses + evidence before fix) and search docs/PATTERNS.md first;
any edit to existing code → T1 (read + call sites);
any non-trivial feature → T4 altitude check + T7 edge grid;
any fix → T3 root-cause gate; any design → T5 least-mechanism order;
multi-step task → T6 assumption ledger; before any commit plan/report → T8 rule refresh.
Before presenting substantial work, self-grade with ~/claude_efficiency/FABLE_BAR_RUBRIC.md.
```

Without this paragraph the protocols are decoration — a model only follows what enters its context. The paragraph is the hook; keep it in every CLAUDE.md.

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` set, layer 2: cognition transfer. These protocols are the written form of what stronger models do unprompted; following them mechanically closes most of the day-to-day gap.*
