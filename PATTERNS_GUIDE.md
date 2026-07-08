# PATTERNS.md — Proven Fixes & Anti-Patterns

> **Purpose of this document:** Guide for maintaining a `docs/PATTERNS.md` in every project. Where DECISIONS.md protects *architecture*, PATTERNS.md protects *implementation*: it is the catalog of solutions paid for in debugging hours, so no future session pays for them again.
>
> **How to use:** Copy the template at the bottom into `docs/PATTERNS.md`. Feed it with the mandatory trigger rule below. This file pairs with the existing bug-log discipline (2+ iteration bugs must be logged) — the bug log records the incident; PATTERNS.md records the reusable lesson.

---

## Why this file exists

Some knowledge can only be bought with iteration: the fix that finally worked after four attempts, the API that behaves differently on device vs. simulator, the innocent-looking configuration that breaks review submission. This knowledge has three properties:

1. It is **invisible in the final code** — the working code doesn't show the four broken attempts.
2. It is **rediscovered at full price** — a new session hits the same wall and burns the same hours.
3. It is **exactly what separates a senior from a junior** on this codebase.

A weaker model with a good PATTERNS.md skips the wall entirely. This is the highest ROI-per-line file in the set, because every entry represents hours already spent that will never be spent again.

## The mandatory trigger (make it a rule, not a habit)

> **Any bug that takes 2+ iterations to fix MUST get a PATTERNS.md entry before the session ends.**

This aligns with the existing bug-log rule. The division of labor:

- **Bug log / session report:** the incident — what happened, in which session, how long it took. Chronological, narrative, for the presentation.
- **PATTERNS.md:** the distilled, *reusable* lesson — symptom, root cause, the fix pattern, the anti-pattern to avoid. Topical, bounded, loaded at session start.

Additional triggers besides 2+ iteration bugs:
- A fix that got **reverted** and redone differently (the first approach is now a documented anti-pattern).
- Any **works-in-sim-fails-on-device** (or dev-vs-prod) discrepancy.
- A **rejection from an external gatekeeper** (App Store review, API deprecation, store policy) and what satisfied it.
- A "**footgun**" discovered in a dependency — behavior that contradicts its documentation or reasonable expectations.

## The entry format

```md
## P-007 — iOS chat composer must use KAV + useHeaderHeight

**Symptom:** Chat input hidden behind / detached from keyboard on iOS;
composer jumps or overlaps content when keyboard animates.
**Failed attempts (do NOT retry these):**
1. Absolute-positioned composer with manual keyboard-height listener —
   breaks on keyboard animation curve changes and on iPad.
2. `KeyboardAvoidingView` alone with `behavior="padding"` — off by exactly
   the navigation header height; looks fine until you scroll.
**Root cause:** KAV measures from the window, not the navigation container,
so the React Navigation header height must be fed in as `keyboardVerticalOffset`.
**The pattern:**
```tsx
const headerHeight = useHeaderHeight();
<KeyboardAvoidingView
  behavior={Platform.OS === "ios" ? "padding" : undefined}
  keyboardVerticalOffset={headerHeight}
>
```
**Scope:** Any screen with a text composer under a React Navigation header.
**Cost of discovery:** ~2 sessions. (GymStride chat, then confirmed again in EXCAR.)
```

### Field-by-field rules

- **ID + title:** Stable `P-NNN` so CLAUDE.md landmines and code comments can point at it (`// see P-007`). Title states the *rule*, not the bug — "composer must use KAV + useHeaderHeight", not "keyboard bug".
- **Symptom:** Written so a future model can *match* it. Include the literal error message, the visual glitch, the exact log line — symptoms are the search keys of this file. A model greps symptoms before it greps code.
- **Failed attempts:** The anti-pattern list. This is the section that saves the hours: it converts "four iterations" into "zero iterations" by making the dead ends visible. Each failed attempt gets its specific failure mode, so the model knows the attempt wasn't just abandoned — it was *disproven*.
- **Root cause:** The mechanism, one paragraph. Without it the pattern is cargo cult; with it, the model can adapt the pattern to a variant of the situation.
- **The pattern:** Real code or exact config. Complete and copy-pasteable — never pseudocode, never abbreviated. (Code is never approximated to save space.)
- **Scope:** Where the pattern applies and where it does *not*. Overgeneralized patterns cause new bugs.
- **Cost of discovery:** The hours/sessions burned. This is not decoration — it tells a future model how much skepticism to apply before deviating from the pattern. "Cost: 2 sessions" means *do not improvise here*.

## Categories worth seeding in every mobile/backend project

Structure the file with these headers from day one; empty sections invite entries:

1. **UI / platform quirks** — keyboard, safe areas, navigation, gesture conflicts, dark mode.
2. **Device vs. simulator vs. production** — every discrepancy discovered.
3. **Store / review gatekeepers** — every rejection (e.g. an ITMS error code) with the exact config that fixed it: which build image, which Info.plist keys, which entitlements. Review rejections cost days of calendar time; their fixes are gold.
4. **Networking & real-time** — WebSocket reconnect/requeue patterns, timeout values that proved correct, ATS/local-network permission recipes.
5. **Build & tooling** — EAS/Xcode/Metro incantations that actually work, port assignments across projects, cache-clearing sequences that resolve "impossible" errors.
6. **Dependency footguns** — library behaviors that contradict their docs.

## How future models should consume this file

Embed in the project's CLAUDE.md:

> When debugging, search `docs/PATTERNS.md` for the symptom FIRST — before proposing fixes. If a pattern matches, apply it as written; do not improvise a variant unless the pattern's Scope excludes your case. When you fix any bug that took 2+ iterations, add a P-entry before ending the session — this is a non-negotiable workflow rule.

## Anti-patterns for the file itself

| Anti-pattern | Why it fails |
|---|---|
| Recording only the working fix | Without failed attempts, the next model retries attempt #1 |
| Vague symptoms ("keyboard was buggy") | Un-greppable; the entry will never be found when needed |
| Pseudocode in the pattern | Model reintroduces the bug while "filling in" the abbreviation |
| No scope boundaries | Pattern gets applied where it doesn't belong → new bugs |
| Mixing incidents with patterns | Incidents belong in session reports; keep this file topical and bounded |
| Entries without IDs | Landmines and code comments can't reference them |

---

## Template (copy from here)

```md
# Patterns — <Project Name>

> Proven fixes and disproven approaches, paid for in debugging hours.
> When debugging: search symptoms here FIRST. When a 2+ iteration bug is
> fixed: add an entry before the session ends (non-negotiable).

## UI / platform quirks

### P-001 — <Rule stated imperatively>
**Symptom:** <Greppable description — literal errors, log lines, visuals.>
**Failed attempts (do NOT retry):**
1. <Approach> — <specific failure mode>.
**Root cause:** <Mechanism, one paragraph.>
**The pattern:**
<complete, runnable code or exact config — never abbreviated>
**Scope:** <Where it applies; where it does NOT.>
**Cost of discovery:** <N iterations / sessions.>

## Device vs. simulator vs. production
## Store / review gatekeepers
## Networking & real-time
## Build & tooling
## Dependency footguns
```

