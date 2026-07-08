# DECISIONS.md — The Decision Log

> **Purpose of this document:** Guide for maintaining a `docs/DECISIONS.md` in every project. This is the file that stops a future (or weaker) model from re-opening settled questions, re-proposing rejected designs, or "improving" something that is deliberately shaped the way it is.
>
> **How to use:** Copy the template at the bottom into `docs/DECISIONS.md`. Add an entry whenever a decision with alternatives is made. Distill from session reports — the session journal is the raw footage; DECISIONS.md is the edited film.

---

## Why this file exists

The single most expensive failure mode of a context-cold model is **relitigating the past**. Symptoms:

- It proposes the architecture you already rejected six weeks ago, with the same arguments.
- It "cleans up" code whose weird shape encodes a hard-won fix.
- It asks the owner questions that were answered in a session the model never saw.
- It flip-flops: session 12 undoes what session 9 built, session 15 restores it.

Code shows *what was decided*. Only a decision log shows *what was rejected and why* — and the rejections are the valuable half. A weaker model with a good decision log will decline to reinvent; a strong model without one will confidently reinvent and be wrong.

## The entry format

Every entry has six fields. The **Rejected alternatives** field is mandatory and is the entire point of the file.

```md
## D-014 — Wi-Fi stays as transport; BLE is pairing-only (2026-07-03)

**Status:** Accepted
**Context:** Device↔phone link needed a transport. BLE looked attractive
(lower power, no network dependency), and a full-BLE design was on the table.
**Decision:** Wi-Fi remains the audio/data transport. BLE is used ONLY for
initial pairing/credential exchange.
**Rejected alternatives:**
- *Full BLE transport* — rejected: bandwidth too low for streaming audio at
  our latency target; iOS background BLE restrictions would break the
  always-listening flow.
- *BLE with audio compression* — rejected: added codec complexity + still
  hit iOS background limits; complexity not justified.
**Consequences:** Device must join user's Wi-Fi during onboarding → onboarding
flow needs the 3-setup-points design. Offline (no-Wi-Fi) use is out of scope.
**Revisit if:** iOS relaxes background BLE, or a device revision adds LTE.
```

### Field-by-field rules

- **ID + title + date:** Stable `D-NNN` ID so other docs (CLAUDE.md landmines, PATTERNS.md, code comments) can reference it as `see D-014`. Title states the decision itself, not the topic — "Wi-Fi stays as transport", not "Transport discussion".
- **Status:** `Accepted` | `Superseded by D-NNN` | `Reversed (date, why)`. Never delete an entry — supersede it. The history of reversals is itself a lesson.
- **Context:** 2–4 lines. What forced the decision. Enough that the entry is understandable in isolation, three months later, by a model that has read nothing else.
- **Decision:** One or two lines, imperative and unambiguous.
- **Rejected alternatives:** Each with its *specific* kill reason. "Rejected: not good" teaches nothing. "Rejected: bandwidth too low for our latency target" lets a future model recognize when the kill reason no longer applies.
- **Consequences:** What this decision forces elsewhere. This is how a model traces "why is the onboarding flow like this?" back to a transport decision.
- **Revisit if:** The escape hatch. Decisions without revisit conditions calcify into dogma; decisions with them stay honest. This line is what elevates a log from bureaucracy to a thinking tool.

## What earns an entry (and what doesn't)

**Log it when:**
- Two or more real alternatives existed and one was chosen (architecture, library, protocol, data model, pricing, scope).
- Something was deliberately *not* done ("no tab bar — dies per meeting 2026-07-03", "no user accounts in v1").
- The owner overruled a model suggestion — capture the owner's reasoning verbatim if possible; this is the highest-value content in the file.
- A previous decision was reversed — log the reversal AND what the failed attempt taught.
- A meeting produced decisions (like the 15-decision EXCAR meeting): each significant one becomes an entry, so meeting outcomes survive beyond the transcript.

**Don't log:**
- Choices with no real alternative (using the platform's standard HTTP client).
- Pure implementation detail reversible in five minutes with no ripple effects.
- Bug fixes — those go to PATTERNS.md unless the *fix strategy* itself was a contested choice.

Threshold heuristic: *"Would a smart newcomer plausibly propose the alternative?"* If yes → log it, because a future model IS that smart newcomer.

## Workflow: session journal → decision log

Selim's projects already keep per-session build journals (`docs/sessions/NN-title.md`). The pipeline:

1. **During the session:** decisions happen inside the work; the session report captures them narratively with the required decision-log-with-rejected-alternatives section.
2. **End of session (2 minutes):** extract each decision that meets the threshold into a `D-NNN` entry in DECISIONS.md. Copy, condense, cross-link: session report gets `→ D-NNN`, entry gets `(see docs/sessions/NN)`.
3. **Never rely on the journals alone.** Journals are chronological and unbounded — a model cannot read 18 session reports at session start. DECISIONS.md is the bounded, load-at-start distillation. Journal = raw footage; DECISIONS.md = the edited film.

## How future models should consume this file

Instructions to embed in the project's CLAUDE.md:

> Before proposing any architectural change, redesign, or "simplification" of an unusual structure, search `docs/DECISIONS.md` for related entries. If your proposal matches a rejected alternative, do not re-propose it unless its stated "Revisit if" condition is now true — and say so explicitly. If you believe a decision is wrong, argue against its *recorded reasoning*, not from a blank slate.

That paragraph converts the log from passive archive into an active guardrail.

## Anti-patterns

| Anti-pattern | Why it fails |
|---|---|
| Entries without rejected alternatives | The alternatives ARE the value; without them it's a changelog |
| "Rejected: worse" | Kill reasons must be specific enough to test later |
| Deleting superseded entries | Loses the reversal history — the most instructive part |
| Logging every tiny choice | Noise buries the 20 decisions that matter; keep the bar high |
| Letting it live only in session reports | Unbounded chronological text is unreadable at session start |
| No IDs | Nothing can reference the decision; knowledge doesn't interlink |

---

## Template (copy from here)

```md
# Decisions — <Project Name>

> Log of decisions with real alternatives. Never delete — supersede.
> Before proposing architectural changes, check here first; don't re-propose
> rejected alternatives unless their "Revisit if" condition is now true.

## D-001 — <Decision stated as a fact> (<YYYY-MM-DD>)

**Status:** Accepted
**Context:** <What forced the choice — 2–4 lines, self-contained.>
**Decision:** <Imperative, unambiguous.>
**Rejected alternatives:**
- *<Alternative>* — rejected: <specific, testable kill reason>.
**Consequences:** <What this forces elsewhere in the system.>
**Revisit if:** <Concrete condition under which this should be reopened.>
```

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` knowledge-distillation set.*
