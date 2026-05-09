# Changelog

All notable changes to the Food Analyzer Skill will be documented here.

Format: `## [version] — YYYY-MM-DD`

---

## [2.0.0] — 2026-05-09

### Breaking changes — major architectural rewrite

The skill is now **condition-agnostic** by default. Hardcoded threshold tables for 8 specific conditions have been removed. Instead, the skill researches authoritative sources at first onboarding and derives a personalized threshold setup, which is then stored locally.

### Removed
- All 8 condition-specific threshold tables (gallbladder, gastritis, T2 diabetes, hypertension, dyslipidemia, weight loss, muscle gain, general healthy eating) from the public `SKILL.md`.
- The condition-specific Adaptive Scoring hierarchy table.
- All gallbladder-flavored examples.

### Added
- **Source Map** — explicit mapping from condition keywords (gallbladder, gastritis, GERD, diabetes, hypertension, dyslipidemia, kidney, IBS, gout, NAFLD, weight loss, muscle gain, healthy eating, etc.) to preferred guideline sources, used at onboarding.
- **Research-based onboarding** — three-step flow: ask user → WebFetch from preferred sources → propose Primary/Secondary parameters with non-overlapping bands → user confirms or refines → save profile.
- **Memory boundary rule** — the skill explicitly does NOT use Claude's broader auto-memory to determine the user's condition. The only canonical state is `~/.claude/skills/food-analyzer/profile.md`.
- **Profile schema upgraded** — profile now stores derived thresholds (Primary parameters with bands, Secondary parameters, Contextual modifiers) and the sources used at onboarding, not just a condition label.
- **Three new generic examples** covering different conditions and modes:
  - Gastritis acute flare → STRICT redirect
  - T2 diabetes → PRAGMATIC menu comparison with explicit suboptimality flag
  - General healthy eating → correction flow showing portion-vs-composition handling
- Expanded source list: KDIGO/NKF, AASLD, ACR/EULAR, Monash FODMAP, ISSN added.

### Changed
- Lifestyle modifiers section made condition-agnostic (no gallbladder-specific phrasing).
- Disclaimer cleanup — full version no longer references gallbladder-specific symptoms.

### Migration
v1.0.0 users with an existing `profile.md` should re-run onboarding to get the new derived-thresholds schema. Old `profile.md` files (with just `condition:` labels) still load but won't have explicit threshold bands — the skill will run a refresh prompt.

---

## [1.0.0] — 2026-05-07

### Added
- Initial public release.
- Bilingual operation — auto-detects user's language (English / Russian) from the first message and replies in the same language.
- **Profile persistence** at `~/.claude/skills/food-analyzer/profile.md` — onboarding runs once, profile loads on every subsequent session (Claude Code only).
- Onboarding: condition, foods to avoid, current phase (acute / remission / prevention).
- **STRICT vs PRAGMATIC** phase modes (set at onboarding, switchable). STRICT redirects bad options; PRAGMATIC ranks but flags suboptimality explicitly.
- **SINGLE MEAL vs DAILY TRACKER** tracking modes. Daily tracker is opt-in, session-scoped, runs a per-day macro budget against condition-specific limits.
- **Deterministic threshold tables** for 8 conditions (gallbladder, gastritis, T2 diabetes, hypertension, dyslipidemia, weight loss, muscle gain, general healthy eating). Non-overlapping value bands, single integer scores. Saturated fat unified to grams per meal across all conditions.
- **Three-tier hierarchy** (Primary / Secondary / Contextual) replaces ad-hoc percentage weights. Cascade rule: any Primary in red → red overall.
- **Per-meal modifier vs Daily Tracker layer separation** — modifiers shift ±1 numeric and never cross color; Daily Tracker is the only override that can shift verdict color, with explicit flag.
- **Range-based macros** with confidence indicator (low / medium / high) and sensitivity disclosure (single biggest swing factor).
- **Visual output** — single-dish output uses a parameter table with score bars (10-char Unicode bars). Multi-dish comparison table includes a fat bar.
- Scoring scale is **1–10** (🟢 8–10 / 🟡 4–7 / 🔴 1–3).
- Parameter row format `Name: 12g — 9/10 🟢` shows actual value and score together to avoid the "is 9/10 fat lots or fine?" ambiguity.
- **Verdict is one sentence** by default. Long clinical reasoning shown only on explicit request ("why", "обоснуй").
- **Evidence levels** on every health claim: 🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate.
- **Conflicting sources rule** — when guidelines disagree (e.g. Stol №5 vs AHA on yolk allowance), surface the conflict instead of silently picking one.
- **Two-tier disclaimer** — short single-line on every red verdict; full version only in STRICT mode or on explicit request.
- Lifestyle modifiers (eating speed, liquid temperature, time of day) marked optional and tagged [⚪ Clinical] — not in default scoring.
- Honesty rules — no score softening, no untagged health claims, no fake authority.
- Correction flow — user can update ingredients, score recalculates honestly.
- Sources — EASL 2016, AHA, ADA, ESC/EAS, DASH/NHLBI, ACG, WHO, USDA FoodData Central, Mayo Clinic, Pevzner №1/№5/№9 (marked as Russian tradition), DGA, Mediterranean.
