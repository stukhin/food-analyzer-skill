# Changelog

All notable changes to the Food Analyzer Skill will be documented here.

Format: `## [version] — YYYY-MM-DD`

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
