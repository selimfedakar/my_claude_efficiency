# claude_efficiency — Knowledge Distillation Set

Guides written at Fable-5 level so that any future model (Opus, Sonnet, or whatever runs next) works on Selim's projects at senior level instead of starting cold. Two layers:

- **Layer 1 — context (6 guides):** general rules and project knowledge. These give a model *background*: what the project is, what was decided, what's forbidden. Necessary, but passive.
- **Layer 2 — cognition (3 files):** **the most important layer, and the one that must actually be applied.** These are *thinking styles* (düşünme şekilleri) written as mandatory procedure — the habits a stronger model performs unprompted, converted into steps a weaker model can follow. Layer 1 tells the model what to know; layer 2 tells it **how to think**. If only one layer gets loaded into a session, load this one.

## Layer 2 — cognition protocols (project-independent, apply everywhere)

| File | What it forces | When it runs |
|---|---|---|
| [THINKING_PROTOCOLS.md](THINKING_PROTOCOLS.md) | Eight protocols T1–T8: read-before-write, 3-hypothesis debugging, root-cause gate, altitude check, least mechanism, assumption ledger, edge-case grid, constraint refresh | During the work — before/while every edit, bug, design |
| [VERIFICATION_PROTOCOL.md](VERIFICATION_PROTOCOL.md) | Definition of done in levels L0–L3, evidence tables with pasted output, regression sweep, Verified/Assumed/Unknown reporting; bans "should work" | Before claiming anything is done |
| [FABLE_BAR_RUBRIC.md](FABLE_BAR_RUBRIC.md) | Self-review as a skeptical senior tester: 8 graded dimensions D1–D8, smell list, pass bar, fix-before-present loop | After drafting, before presenting to the owner |

These three only work if they enter the model's context: paste the **loading paragraph** (bottom of THINKING_PROTOCOLS.md) into every project's CLAUDE.md — that paragraph is the hook that makes a session actually read and obey them.

## Layer 1 — context guides (produce per-project brain files)

| # | Guide | Produces (per project) | Protects |
|---|---|---|---|
| 1 | [CLAUDE_MD_GUIDE.md](CLAUDE_MD_GUIDE.md) | `CLAUDE.md` | The present: commands, map, landmines, owner rules |
| 2 | [DECISIONS_GUIDE.md](DECISIONS_GUIDE.md) | `docs/DECISIONS.md` | The past: choices + rejected alternatives |
| 3 | [PATTERNS_GUIDE.md](PATTERNS_GUIDE.md) | `docs/PATTERNS.md` | Paid lessons: proven fixes, disproven attempts |
| 4 | [SKILLS_GUIDE.md](SKILLS_GUIDE.md) | `.claude/skills/*/SKILL.md` | Repeating workflows as executable recipes |
| 5 | [REVIEW_CHECKLIST_GUIDE.md](REVIEW_CHECKLIST_GUIDE.md) | `docs/REVIEW-CHECKLIST.md` | The quality bar, frozen into binary checks |
| 6 | [ROADMAP_GUIDE.md](ROADMAP_GUIDE.md) | `docs/ROADMAP.md` | The future: direction, deferrals, anti-goals |

## Rollout order for an existing project

0. **Layer 2 hook first** — paste THINKING_PROTOCOLS.md's loading paragraph into the project's CLAUDE.md. Zero mining required, immediate effect on every subsequent session.
1. **CLAUDE.md** — highest-leverage layer-1 file, loaded every session.
2. **PATTERNS.md** — mine the existing session reports for every 2+ iteration bug.
3. **DECISIONS.md** — mine session reports + meeting notes for decisions with alternatives.
4. **ROADMAP.md** — one planning pass with the owner.
5. **REVIEW-CHECKLIST.md** — copy the master list, add the project section from steps 1–2.
6. **Skills** — last; they depend on the rules the other files pin down.

## What each layer buys (honest scope)

Layer 1 closes the **context gap**: a weaker model stops relitigating decisions, re-hitting solved bugs, and breaking owner rules. Layer 2 closes the **discipline gap**: it stops first-idea fixes, unverified "done ✅" claims, symptom patches, and happy-path testing — which is most of the day-to-day quality difference between model tiers. What no file can transfer: raw insight on genuinely novel problems. Everything short of that is transferable, and these files are the transfer.

## The one-line summary of the whole method

> Strong models have taste and judgment; weak models follow instructions well. Distillation = converting taste into instructions while the strong model is still here.

*Authored by Claude Fable 5 (claude-fable-5) — 2026-07-07.*
