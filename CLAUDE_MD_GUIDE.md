# CLAUDE.md — The Project Brain File

> **Purpose of this document:** This is the master guide for writing a `CLAUDE.md` file for any project. A `CLAUDE.md` is the single most leveraged file in a repository: it is loaded into the model's context at the start of *every* session, so every word in it either pays rent or wastes tokens. A strong CLAUDE.md lets a weaker or context-cold model behave like one that has worked on the project for months.
>
> **How to use:** Copy the template at the bottom into a new project's root as `CLAUDE.md`, then fill it using the section-by-section guidance below. Revisit it at the end of every significant session — a stale CLAUDE.md is worse than none, because the model will trust it.

---

## Core philosophy

A model reading your repo cold can figure out *what* the code does. What it cannot figure out is:

1. **Intent** — why the code is shaped this way, what it is trying to become.
2. **Landmines** — the things that look safe to change but aren't.
3. **Your rules** — workflow constraints that live in your head, not in the code.
4. **The map** — which 10 of the 400 files actually matter.

CLAUDE.md exists to transfer exactly these four things. It is *not* documentation for humans, not a README, not marketing. Write it for a very capable colleague who has amnesia and infinite patience but zero context.

### The three laws of a good CLAUDE.md

1. **Every line must change behavior.** If removing a line wouldn't change what the model does, remove it. "This project uses React" is useless if the package.json says so. "We use React 18 but must NOT use Server Components because Expo doesn't support them" changes behavior.
2. **Prefer prohibitions and constraints over descriptions.** Models are good at reading code, bad at knowing what they're not allowed to touch. "Never edit `generated/` — it's overwritten by codegen" is worth 50 lines of architecture prose.
3. **Absolute over relative.** Never write "recently", "the new API", "the old way". Write dates, versions, file paths. A weaker model cannot resolve "recently" — it will guess wrong.

---

## Section-by-section guide

### 1. One-paragraph identity

The first paragraph must answer: what is this, who is it for, what stage is it at, and what is the current top priority. This calibrates *everything* the model does afterward — a prototype gets different engineering rigor than a shipped App Store app.

**Good example:**
> GymStride is a live iOS App Store app (social gym-streak tracker, React Native + Expo, Railway backend). It has real users. Stability > features. Current focus: v1.1 (Sentry, streaks UI). Never break existing user data or push schema migrations without an explicit plan.

**Bad example:**
> This is a fitness app built with modern technologies.

### 2. Commands block

Exact, copy-pasteable commands for: install, run dev, run tests, typecheck, lint, build, deploy. Include ports, env vars, and *which directory to run them from* in a monorepo. This is the highest-frequency section — the model uses it every session.

```md
## Commands (run from repo root unless noted)
- Backend dev: `PYTHONPATH=src uvicorn app.main:app --reload --port 8000`
- Mobile dev:  `cd mobile && npx expo start --port 8084`  ← 8082 is taken by another project
- Tests:       `cd backend && pytest` (363 tests, must stay green)
- Typecheck:   `cd mobile && npx tsc --noEmit`
```

Note the inline `←` annotation: constraints attached directly to the command they constrain. That is the pattern — don't put the port rule three sections away.

### 3. Architecture map (the 80/20 map, not a full tree)

Do NOT dump a file tree. List only the files/directories where real decisions live, with one line each on their role and one line on what depends on them. 10–20 entries max. The model can `ls` the rest.

```md
## Where things live
- `src/app/routers/actions.py` — WebSocket instant-action channel. Requeue-safe by design; do not "simplify" the requeue logic.
- `mobile/src/state/connection.ts` — the connection state machine. ALL connectivity UI derives from this; never add ad-hoc `isConnected` booleans elsewhere.
- `shared/endpointing.ts` — endpointing policy shared between backend and mobile. Change it in one place only; both sides import it.
```

Each entry pairs *location* with *constraint*. Location alone is discoverable; the constraint is not.

### 4. Data flow in one paragraph

One paragraph tracing the main request/event through the system: "Mic audio → WS `/stream` → VAD → STT → prompt tree → tool dispatch → TTS → device speaker; instant actions bypass TTS via `/actions/ws`." A weaker model with this paragraph makes correct guesses about where a bug lives; without it, it greps blindly.

### 5. Landmines / "Do NOT" list

The most valuable section. Every entry has the shape: **prohibition + reason + what to do instead.** The reason is mandatory — a model that knows *why* generalizes the rule correctly; a model with a bare rule violates its spirit.

```md
## Landmines
- Do NOT use absolute positioning for chat composers. It breaks with the keyboard on iOS.
  Use `KeyboardAvoidingView` + `useHeaderHeight()` — the ONLY pattern that works here (cost us 2 sessions to learn).
- Do NOT run `git commit` or `git push`. Ever. Hand the exact commands to Selim; he runs them. Commits carry no AI signature.
- Do NOT rename anything in `AppShortcuts.swift` — names are user-facing Siri phrases; renaming breaks users' existing shortcuts.
```

Feed this section from real pain: every bug that took 2+ iterations to fix, every reverted change, every "no, put it back" from the owner becomes a landmine entry.

### 6. Workflow rules (owner's laws)

The non-negotiables of *how* work happens in this repo, distilled:

```md
## Workflow rules (non-negotiable)
- Atomic commits: ONE file per commit. Prepare a numbered commit plan; owner executes it.
- All source files, comments, and docs in English. Chat can be Turkish.
- Every session ends with a build journal entry: `docs/sessions/NN-title.md` (see DECISIONS.md guide for format).
- Bugs that take 2+ iterations get a root-cause entry in PATTERNS.md before the session ends.
```

### 7. Environment quirks

Machine-specific facts a model cannot infer: which ports are reserved by *other* projects, simulator vs. device gotchas, credentials locations (never the secrets themselves — only where they live), Xcode signing team, deployment targets.

### 8. Current state snapshot (dated!)

A short, **dated** section: what's deployed, what's pending, what's known-broken. Because it goes stale fastest, it carries a date and an explicit instruction:

```md
## State as of 2026-07-07 (verify before trusting — check git log and deploy dashboard)
- Backend deployed: version c0df2644 (rate limiting live, fail-closed auth verified)
- Pending: physical BLE test, ASC app record for eas.json ascAppId
- Known broken: nothing
```

### 9. Pointers to the sibling brain files

```md
## Deeper context
- Past decisions & rejected alternatives → docs/DECISIONS.md
- Proven fixes & anti-patterns → docs/PATTERNS.md
- Review standards → docs/REVIEW-CHECKLIST.md
- Direction & deliberate deferrals → docs/ROADMAP.md
```

---

## Anti-patterns (what makes CLAUDE.md fail)

| Anti-pattern | Why it fails |
|---|---|
| Pasting the README | Model already infers this from code; wastes the context budget |
| Full file tree dump | Noise drowns the 10 files that matter; model can `ls` |
| Vague adjectives ("clean code", "be careful") | Not actionable; changes nothing |
| Undated state claims | Model trusts stale facts confidently — worse than ignorance |
| Rules without reasons | Weaker model obeys the letter, violates the spirit |
| One giant file >300 lines | Push overflow into DECISIONS/PATTERNS; CLAUDE.md is the index, not the archive |

## Maintenance ritual

At the end of every meaningful session, ask three questions:
1. Did I learn a constraint the hard way this session? → Landmines section.
2. Did any command/port/path change? → Commands section.
3. Did deployed state change? → State snapshot, with today's date.

Sixty seconds of maintenance per session keeps the file alive. An unmaintained CLAUDE.md decays into a trap.

---

## Template (copy from here)

```md
# <Project Name>

<One-paragraph identity: what, for whom, what stage, current priority, rigor level.>

## Commands (run from <dir> unless noted)
- Install: `...`
- Dev: `...`  ← port/env constraints inline
- Tests: `...` (N tests, must stay green)
- Typecheck/lint: `...`
- Build/deploy: `...`

## Where things live
- `path/to/file` — role. Constraint.
(10–20 entries max)

## Data flow
<One paragraph tracing the main request through the system.>

## Landmines
- Do NOT <X>. <Why.> Instead: <Y>.

## Workflow rules (non-negotiable)
- <Owner's laws: commits, language, journals, git policy.>

## Environment quirks
- <Ports reserved by other projects, signing, simulators, where secrets live.>

## State as of <DATE> (verify before trusting)
- Deployed: ...
- Pending: ...
- Known broken: ...

## Deeper context
- docs/DECISIONS.md · docs/PATTERNS.md · docs/REVIEW-CHECKLIST.md · docs/ROADMAP.md
```

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` knowledge-distillation set: high-capability model writes the thinking down so any future model can pick it up.*
