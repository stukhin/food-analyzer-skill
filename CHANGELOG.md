# Changelog

All notable changes to the Food Analyzer Skill will be documented here.

Format: `## [version] — YYYY-MM-DD`

---

## [2.1.0] — 2026-05-09

### Added (nine refinements from real-use testing)

- **Anti-disorder safeguards (Honesty Rule 8)** — new `ED-safe mode` flag in the profile. Triggered at onboarding by a history-of-eating-disorder question, or surfaced in-session if the user shows obsessive macro tracking / restrictive escalation / body-shame language. Effects: numbers given as ranges instead of point values, precision dropped below 5g / 10kcal, restrictive framing replaced with balance-focused language, professional referral on red flags.
- **Realistic-modifications rule (Honesty Rule 9)** — modifications must be things a kitchen can actually do without breaking the dish. No "no cheese in carbonara", no "skip the cream in alfredo", no "no rice in risotto". If the only honest mod is "order something else", say that.
- **Per-parameter confidence** — replaced single `Уверенность: medium` with three-part breakdown `макро [H/M/L] · готовка [H/M/L] · порция [H/M/L]`. Each component is independent — you can have high confidence on cooking method but low on portion size.
- **Personal calibration log** — new section in profile.md. Skill appends entries when the user volunteers post-eating feedback ("после этого было плохо", "хорошо зашло"). After 3+ consistent entries about the same trigger, skill surfaces it and offers to update profile restrictions.
- **Religious / ethical / cultural restrictions** — onboarding now explicitly asks about dietary pattern (vegan / vegetarian / pescatarian / halal / kosher / none) and personal aversions, separate from medical avoidances. Profile schema split: `Medical / allergies`, `Dietary pattern`, `Personal aversions`.
- **Phase priority for conflicting conditions** — new Step 1.5 in onboarding. If user names multiple conditions (NAFLD + dyslipidemia, etc.), skill asks which is the priority right now. Priority condition → Primary parameters; others contribute to Secondary.
- **One-time onboarding disclaimer (Step 0)** — before any questions, skill shows a clear "I'm a tool, not medical advice" disclaimer and waits for acknowledgement. Replaces the previous pattern where the disclaimer only appeared on red verdicts.
- **Profile staleness check** — on load, if profile is older than 6 months, skill asks once "освежить онбординг или работаем с текущим?" before scoring.
- **Auto-backup before profile rewrite** — every profile change (re-onboarding, phase switch, calibration update, restriction change) creates `~/.claude/skills/food-analyzer/backups/profile.md.backup-YYYYMMDD-HHMMSS`. Last 5 backups retained, `backups/` is gitignored.

### Changed

- Onboarding is now four steps (Step 0 disclaimer → Step 1 questions → Step 1.5 priority pick if multi-condition → Step 2 research → Step 3 confirm).
- Profile schema restructured (Identity / Restrictions / Sources / Derived thresholds / Personal calibration log) — old single-line Identity preserved as compatibility but new format takes precedence.
- Example outputs in SKILL.md updated to use new confidence format and Rule 9-compliant modifications.

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
