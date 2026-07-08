# .claude/skills/ — Executable Recipes for Repeating Work

> **Purpose of this document:** Guide for writing Claude Code *skills* (slash commands) that encode a repeating workflow as a step-by-step recipe. A skill is the strongest form of knowledge distillation: CLAUDE.md tells a model what to know, DECISIONS/PATTERNS tell it what to avoid — a skill tells it *exactly what to do*, so even a much weaker model executes the workflow at full quality by following instead of thinking.
>
> **How to use:** Create files under `<project>/.claude/skills/<name>/SKILL.md` (project-scoped) or `~/.claude/skills/<name>/SKILL.md` (available everywhere). Invoke as `/<name>` in Claude Code. Templates and two worked examples below.

---

## When a workflow deserves a skill

Turn it into a skill when **all three** are true:

1. **It repeats** — done 3+ times, or will be done every session/release.
2. **It has a fixed correct sequence** — order matters, steps are checkable.
3. **Deviating is expensive** — a skipped step breaks a build, violates an owner rule, or ships a mistake.

If a workflow is done twice and both times the model needed correcting — that's the signal. The correction you gave IS the skill content.

**Prime candidates from real workflows:**
- Session-end ritual (report + memory + commit plan) — every session, fixed sequence, owner rules involved.
- TestFlight release (build → submit → verify) — rare enough to forget, expensive to get wrong.
- Atomic commit plan generation — a strict owner rule with a mechanical procedure.
- Deploy + post-deploy verification — mistakes here hit production.

**Not skill material:** one-off tasks, exploratory/creative work, anything whose steps change every time. A skill for "fix the bug" is meaningless; a skill for "release to TestFlight" is gold.

## Anatomy of a strong skill

A skill file is markdown with YAML frontmatter. The body is instructions the model follows *verbatim* when the skill is invoked.

```md
---
name: session-report
description: Write the end-of-session build journal entry and update project docs. Use at the end of every working session.
---

<step-by-step instructions here>
```

### The seven rules of skill-writing

1. **Frontmatter `description` is the trigger.** The model decides *whether* to use a skill from its description alone. Include when-to-use language: "Use at the end of every working session", "Use when the user asks for a TestFlight build".

2. **Numbered steps, one action each.** A weaker model executing a skill is a pilot on a checklist. "Step 3: Run `npx tsc --noEmit` from `mobile/`. If it fails, STOP and report errors — do not proceed to step 4."

3. **Exact commands, exact paths, never placeholders that require judgment.** `<version>` that the model must compute is fine *if* the computation is spelled out ("version = highest NN in docs/sessions/ + 1"). `<do the appropriate thing>` is a bug.

4. **Gates between phases.** After any step that can fail (build, test, upload), an explicit gate: what output means success, what to do on failure. Weaker models plow through failures unless told to stop — gates are where recipe-following beats improvisation.

5. **Embed the owner's rules inline where they bite.** Don't rely on CLAUDE.md being remembered mid-workflow: "Step 7: Output the commit plan as commands for Selim to run. Do NOT run git commit yourself — hard rule, all projects."

6. **Include the output format literally.** If the skill produces a document (session report, commit plan), embed the exact skeleton in the skill body. Format drift disappears when the format is in the recipe.

7. **End with a self-check.** Final step = verification checklist: "Confirm: (a) report file exists and is in English, (b) README row added, (c) memory updated, (d) no git commands were run. Report each with ✅/❌."

## Worked example 1: `/session-report`

```md
---
name: session-report
description: Write the end-of-session English build journal entry (docs/sessions/NN-title.md), update the README index, and produce the atomic commit plan. Use at the end of every working session in this project.
---

You are ending a working session. Execute these steps in order.

1. Determine the report number: list `docs/sessions/`, find the highest
   `NN` prefix, add 1. Zero-pad to two digits.

2. Review this session's conversation and produce the report at
   `docs/sessions/NN-<kebab-title>.md`, in English, with EXACTLY these sections:

   # Session NN — <Title>  (<YYYY-MM-DD>)
   ## What was built
   <each change: what + why + how + purpose>
   ## Bugs encountered
   <for each: symptom, root cause, fix; flag any 2+ iteration bug with **[PRESENTATION]**>
   ## Decision log
   <each decision WITH rejected alternatives and why they were rejected>
   ## State at session end
   <deployed versions, pending items, known broken — dated>
   ## Next session
   <concrete first actions>

3. For every 2+ iteration bug in section "Bugs encountered": add or update a
   P-entry in docs/PATTERNS.md (format defined at the top of that file).

4. For every entry in "Decision log" that had real alternatives: add a D-entry
   to docs/DECISIONS.md.

5. Add one row for this report to the sessions table in README.md.

6. Update the "State as of <DATE>" section of CLAUDE.md with today's date.

7. Produce the atomic commit plan: one commit per file, ordered so the repo
   builds after every commit. Output as a numbered list of exact
   `git add <file> && git commit -m "<message>"` commands FOR THE OWNER TO RUN.
   Do NOT execute any git command yourself — hard rule, all projects.

8. Self-check — report each with ✅/❌:
   (a) report exists, English, all 5 sections present
   (b) PATTERNS/DECISIONS updated or explicitly "no entries this session"
   (c) README row added
   (d) CLAUDE.md state section re-dated
   (e) zero git commands executed by you
```

## Worked example 2: `/testflight` (compressed)

```md
---
name: testflight
description: Build and submit the iOS app to TestFlight via EAS, then verify processing. Use when the user asks for a TestFlight build or release.
---

1. Preflight — verify ALL before building (STOP and report if any fails):
   - `git status` clean or owner has confirmed uncommitted state is intended
   - version/buildNumber bumped in app.json (show owner old → new)
   - `npx tsc --noEmit` passes; test suite passes
   - eas.json: correct Xcode image and ascAppId present (see P-entries for
     past store rejections — confirm their fixes are still in place)
2. Build: `eas build --platform ios --profile production`
   Gate: wait for SUCCESS; on failure, paste the failing step's log and STOP.
3. Submit: `eas submit --platform ios --latest`
4. Verify: check App Store Connect processing status; warn owner that
   missing-compliance/metadata issues appear only after processing.
5. Report: build number, submission id, and a checklist of what the owner
   must still do by hand (external tester invites, export compliance answer).
```

## Distillation strategy: burning strong-model judgment into recipes

The reason to write skills *now*, with a strong model: every place a workflow says "judge / decide / choose appropriately", a strong model decides well and a weak model decides poorly. Skill-writing is the act of **replacing judgment calls with predecided rules**:

- Weak: "write a good commit message" → Strong, predecided: "message = `<area>: <imperative change>`, ≤ 60 chars, no AI signature, no period".
- Weak: "check the build looks okay" → Strong, predecided: "SUCCESS status AND artifact URL present AND no warnings containing `ITMS`".

Walk each workflow once with the strong model, capture every micro-decision it makes, and write those decisions into the skill as rules. That is the transfer.

## Maintenance

- A skill that produced a wrong result gets fixed **the same day** — a broken recipe is worse than none, because it's followed with confidence.
- When PATTERNS.md gains an entry that affects a workflow (e.g. a new store-rejection fix), update the corresponding skill's preflight in the same session.
- Version-sensitive commands (EAS profiles, Xcode images) should say where the canonical value lives rather than hardcoding twice: "use the image pinned in eas.json — do not override".

---

*Authored by Claude Fable 5 (claude-fable-5) for Selim Fedakar — 2026-07-07. Part of the `claude_efficiency` knowledge-distillation set.*
