# ROADMAP.md — The Intent File

> **Purpose of this document:** Guide for maintaining a `docs/ROADMAP.md` in every project. This is the file that transfers *direction*: where the project is going, what is deliberately deferred, and what will never be built. Code shows the present; DECISIONS.md shows the past; ROADMAP.md shows the intended future — and a model that knows the intended future makes structurally better decisions in the present.
>
> **How to use:** Copy the template at the bottom into `docs/ROADMAP.md`. Update at every milestone and after every planning meeting. Keep it under two pages — it is a compass, not a backlog.

---

## Why this file exists

Without a roadmap, a model optimizes for the current task in isolation. That produces locally-good, globally-wrong choices:

- It hardcodes single-user assumptions the week before multi-user work begins.
- It builds elaborate infrastructure for a feature the owner decided to kill.
- It "helpfully" adds the deferred feature half-way, creating a broken stub the real implementation must fight.
- It cannot triage: asked to fix three things with time for one, it picks by ease instead of by strategic weight.

The roadmap fixes all four by answering: *what matters next, what is postponed on purpose, and what is out of bounds.*

## The five sections

### 1. North star (one paragraph)

What the project is trying to become and for whom, stated concretely enough to arbitrate real trade-offs. Not a slogan — a decision tool.

> **Weak:** "Auris will revolutionize personal audio AI."
> **Strong:** "Auris = wearable + phone app where the device handles capture and the phone handles intelligence; wins on latency and privacy, not feature count. Anything adding cloud round-trips to the hot path works against the product."

The strong version lets a model *reject* a feature idea by itself: "adds a cloud round-trip to the hot path → against north star → flag instead of build."

### 2. Now / Next / Later

Three buckets, each item one line with its *why*. Never more than ~5 items per bucket — a 30-item bucket is a backlog wearing a roadmap costume.

```md
## Now (current milestone — v1.1, target: 2026-08)
- Sentry crash reporting — we're blind to production crashes; blocks everything else
- Streak UI rework — most-requested user complaint
## Next (after v1.1 ships)
- Pro subscription + HealthKit (v1.2) — revenue before feature expansion
## Later (intent exists, no commitment)
- Android — only after iOS retention proves the model
```

**Rule for models:** items in *Later* must not leak into current code as speculative abstraction. Do not build "flexibility for Android" while shipping the iOS milestone — that flexibility is a bet the owner hasn't placed yet.

### 3. Deliberately deferred (the "not yet" list)

The highest-value section for a model. Each entry: what, why deferred, and what would un-defer it. This is the difference between "missing" (model should flag or fix) and "postponed" (model must NOT fix).

```md
## Deliberately deferred — do NOT build these prematurely
- Physical-device BLE test path: deferred until the hardware is at hand (USB access needed).
  NOT a gap in the code — do not stub or simulate it further. Un-defers: hardware on desk.
- Caregiver network feature: idea captured (see project notes), zero implementation on purpose —
  validate core retention first. Un-defers: owner's explicit go.
- RevenueCat webhook: known revenue leak, accepted short-term. Un-defers: v1.1 shipped.
```

Without this section, every capable model that touches the repo will notice these "gaps" and burn a session (or worse, ship a half-feature) fixing what was postponed on purpose.

### 4. Never list (anti-goals)

Things decided *against*, permanently or until explicitly revisited — with the reason, cross-referenced to DECISIONS.md entries where they exist.

```md
## Never (do not propose, do not build)
- Tab bar navigation — killed in the 2026-07-03 meeting (see D-0XX); single-surface design is the product.
- AI/Claude attribution in commits, comments, or UI strings — owner rule, all projects.
- Features requiring always-on cloud audio storage — against the privacy positioning.
```

A "never" without a reason gets eroded by a helpful model ("surely they'd want this now"). A "never" with a reason holds.

### 5. Open questions (parked, with owners)

Questions that shape the future but aren't answered yet — parked *visibly* so a model neither answers them unilaterally nor forgets they exist.

```md
## Open questions (do not resolve unilaterally — surface when relevant)
- Pricing model (one-time vs. subscription)? — owner decision, blocks store metadata.
- web-2 → main branch reconciliation strategy? — pending team agreement; do NOT blind-merge.
```

## How future models should consume this file

Embed in the project's CLAUDE.md:

> Read `docs/ROADMAP.md` before proposing features, adding abstraction "for the future", or triaging what to do first. Rules: (1) don't build *Later* items or speculative flexibility for them; (2) never touch *Deliberately deferred* items except to keep them compiling; (3) never propose *Never* items unless you present a changed condition that defeats the recorded reason; (4) when a task conflicts with the north star, say so before implementing.

## Relationship to the other brain files

| File | Time axis | Question it answers |
|---|---|---|
| CLAUDE.md | Present | How do I work here today? |
| DECISIONS.md | Past | What was chosen and what was rejected? |
| PATTERNS.md | Past (paid lessons) | What is proven / disproven in this code? |
| ROADMAP.md | Future | What matters next, and what is off-limits? |
| REVIEW-CHECKLIST.md | Gate | Does this change meet the bar? |

An item often flows: Open question → decision meeting → D-entry in DECISIONS.md → its consequence lands in Now/Never here → its landmine lands in CLAUDE.md. Keeping the flow explicit is what makes five files behave like one brain.

## Maintenance

- Update at every milestone boundary and after every planning meeting — the meeting notes are the raw input; the roadmap is the distillate.
- Date the "Now" section's milestone target; stale targets are demoralizing and misleading — move or re-date honestly.
- When a deferred item un-defers, move it to Now/Next and note the trigger fired. When a Never gets revisited, that's a new D-entry, not a silent edit.

---

## Template (copy from here)

```md
# Roadmap — <Project Name>
> Compass, not backlog. Read before proposing features or adding abstraction.
> Last updated: <YYYY-MM-DD>

## North star
<One concrete paragraph that can arbitrate trade-offs.>

## Now (<milestone>, target <date>)
- <item> — <why it's first>
## Next
- <item> — <why it follows>
## Later (intent, no commitment — do NOT pre-build flexibility for these)
- <item> — <what would promote it>

## Deliberately deferred — do NOT build prematurely
- <item>: <why deferred>. Un-defers: <concrete trigger>.

## Never (do not propose without defeating the recorded reason)
- <item> — <reason / D-NNN reference>.

## Open questions (surface, don't resolve unilaterally)
- <question> — <who decides, what it blocks>.
```

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` knowledge-distillation set.*
