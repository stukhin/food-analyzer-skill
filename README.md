# 🥗 Food Analyzer Skill for Claude

A Claude skill that analyzes food photos, menu screenshots, and dish descriptions to evaluate nutritional content and health suitability — personalized to your dietary goals or medical condition. Works in English and Russian (replies in whichever language you start the conversation in).

**v3.0 — knowledge-base architecture.** The skill is no longer a single file: it's a structured set of modules (methodology + personal profile + cuisine knowledge base + cached evidence + auto-backups).

---

## What it does

- **Personalizes at onboarding** — five-group research-based wizard. You describe your condition or goal in your own words; the skill researches authoritative guidelines (EASL, AASLD, ADA, AHA, ACG, DASH/NHLBI, ESC/EAS, KDIGO, ACR, AASLD, Monash FODMAP, DGA, etc.), proposes priority parameters with non-overlapping value bands, you confirm or refine.
- **Handles multiple conditions** — if you have several (e.g. NAFLD + dyslipidemia), the skill asks which is the priority right now and structures Primary vs Secondary parameters accordingly.
- **Respects religious / ethical / cultural restrictions** — halal, kosher, vegan, vegetarian, pescatarian, plus personal aversions, separate from medical avoidances.
- **Analyzes food photos** — identifies ingredients, estimates КБЖУ / macros as ranges, with **four-aspect confidence breakdown** (macros / cooking / portion / sauce-composition).
- **Reads menu screenshots** — multi-dish comparison table with visual fat bars and explicit "best / worst / what to order" sections.
- **Scores each dish** on a 1–10 scale with a green/yellow/red stoplight, looked up in your personally-derived threshold tables.
- **Tags every health claim** with an evidence level (🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate) — no fake authority.
- **Applies reusable evaluation heuristics** — healthy-sounding traps (Caesar, Cobb, "wellness bowls"), hidden-fat sources (mazesoba oil pool, mayo "special sauces", coconut milk), broth fork (clear vs creamy), cut/protein ladder, cooking-method ladder.
- **At-the-table behavioral library + mandatory re-scoring delta** — for 🟡/🔴 dishes, shows concrete modifications and the new score: *"as served 4/10 🟡 → modified 7/10 🟢"*.
- **Suggests only realistic modifications** — no "ask for carbonara without cheese". If the only honest mod is "order something else", it says that.
- **Indulgence framework (PRAGMATIC mode only)** — for treats, doesn't answer binary yes/no. Stacked risk modulators (portion, timing, personal aids, frequency, post-meal activity), red-flag combinations, lighter substitutes.
- **Wellness-label skepticism** — "sugar-free / natural / protein / wellness / lite / clean / plant-based" never auto-raises a score.
- **Drinks hierarchy** — six-tier ladder; explicitly distinguishes coconut water (light) from coconut milk (saturated fat heavy).
- **Quality-aware modifiers** — fat type matters as much as quantity (olive oil neutral vs butter/cream −1 vs lard −2). Sugar source matters (added vs natural vs sugar alcohols).
- **ED-safe mode** — if your history includes an eating disorder, the skill softens output (ranges, no gram-level precision, balance-focused framing) and refers to a professional on red flags.
- **Personal calibration log** — when you say "это плохо зашло" or "хорошо после этого", the skill appends to a log. After 3+ consistent entries on the same trigger, it surfaces the pattern and offers a profile update.
- **Tracks daily budget** (opt-in) — per-day macro budget with verdicts that shift in context.
- **Meal Planning (Mode C, minimal in v3.0)** — day or week structured plan with running totals; full mechanics in v3.1.
- **Persists your profile** — onboarding runs once; reads back on every future session (Claude Code). Profile is local — never committed to git.
- **Auto-backs up your profile** — every change creates a timestamped backup; last 5 retained.
- **Staleness check** — if profile is older than 6 months, the skill asks whether to refresh.
- **Memory-boundary safe** — does NOT pull condition info from Claude's broader auto-memory; works only on explicitly-confirmed onboarding data.
- **Accepts corrections** — tell it what it got wrong, it recalculates honestly.

---

## Knowledge-base layout (v3)

```
food-analyzer/
├── SKILL.md                    # methodology only; zero personal data
├── profile.md                  # your personal data + derived thresholds (gitignored)
├── cuisines/
│   ├── _template.md            # blank template for adding new cuisines
│   ├── italian.md
│   ├── japanese.md
│   ├── thai.md
│   ├── mediterranean.md
│   ├── american.md
│   └── french.md
├── references/
│   ├── _sources.md             # registry of 19 authoritative bodies
│   └── guidelines_cache.md     # cached guideline summaries (90-day refresh)
└── backups/                    # auto-snapshots of profile.md before any edit
```

`profile.md` and `backups/` are gitignored — your personal data stays local.

---

## Key principle

This skill is intentionally critical. It will not soften bad scores to make you feel better.

- **Red is red.** No "best of bad options" in STRICT mode (acute / post-op / diagnostic).
- In PRAGMATIC mode (remission / general life), ranking is allowed — but if the top option is yellow or worse, the skill explicitly says so and describes what a truly green choice would look like.
- **Modifications must be physically realistic** — Rule 9: no "carbonara without cheese".
- **Indulgence framework only in PRAGMATIC** — Rule 11: STRICT means redirect, not risk-modulate.
- Scoring is deterministic: every threshold table uses non-overlapping value bands mapped to single integer scores. No vibes.

---

## Install

### Claude Code (personal skill)

```bash
git clone https://github.com/stukhin/food-analyzer-skill.git ~/.claude/skills/food-analyzer
```

Restart Claude Code (or start a new session). The skill activates automatically when you describe food or send a photo.

### Claude.ai (manual)

Copy `SKILL.md` and paste it at the start of a conversation as a system instruction, then send your photo or dish description. Note: Claude.ai web does not preserve profile.md across sessions (no file system access) — onboarding runs each time.

---

## How to use

**First session**: Claude shows a disclaimer, then runs the five-group wizard (basics / health context / goals / restrictions / personal calibration & ED screening). For each condition you name, it researches the corresponding guidelines (via WebFetch in Claude Code) and proposes derived thresholds. You confirm or refine. Profile is saved to `~/.claude/skills/food-analyzer/profile.md`.

**Subsequent sessions**: Profile loads automatically. Skill confirms it once and skips onboarding (unless older than 6 months — then asks to refresh).

**Inputs**:
- Send a photo of food → table-format breakdown + verdict + modifications with re-scored delta
- Send a menu screenshot → comparison table with fat bars + "топ / худшее / что заказать"
- Describe a dish in text → estimated analysis
- Say *"веду дневник"* / *"track my day"* → activates Daily Tracker mode
- Ask *"can I have [treat] sometimes?"* in PRAGMATIC mode → indulgence framework response
- Say *"что съесть сегодня"* / *"plan my week"* → Mode C meal plan

---

## Scoring

| Score | Meaning |
|-------|---------|
| 🟢 8–10 | Safe — good choice for your condition |
| 🟡 4–7 | Caution — eat in moderation or modify |
| 🔴 1–3 | Avoid — specific reason always provided |

Each parameter shows the actual value alongside the score, e.g. `Fat: 12g — 9/10 🟢`, to avoid the ambiguity of "is 9/10 fat lots or fine?"

Parameters scored per dish (depends on your condition):
- Total fat per meal (g)
- Saturated fat per meal (g) — uniform across all conditions
- Cholesterol, sodium, sugar, fiber, protein, carbs as relevant
- Cooking method as a modifier
- Quality adjustments (fat type, sugar source)
- Variable-size modifiers (omega-3 bonus, fat-shock penalty, etc.)
- Overall weighted via Primary / Secondary / Contextual hierarchy

---

## Supported conditions

The skill is condition-agnostic — there are no hardcoded condition tables in `SKILL.md`. At first onboarding, you describe your condition or goal in your own words and the skill matches it to a row in its **Source Map** (in `SKILL.md`):

- Gallbladder / biliary / gallstones → EASL Gallstone Guidelines, Mayo Clinic, Stol №5
- Gastritis / peptic ulcer / GERD → ACG, Mayo Clinic, Stol №1
- Diabetes (T1, T2, glycemic control) → ADA Standards of Care, Mayo Clinic
- Hypertension → DASH / NHLBI, AHA
- Dyslipidemia / high cholesterol / high LDL → ESC/EAS Dyslipidaemia, AHA
- Kidney / CKD → KDIGO, NKF, Mayo Clinic
- IBS → Monash FODMAP, ACG IBS, NICE
- Gout → ACR, EULAR, Mayo Clinic
- NAFLD / fatty liver → AASLD, EASL NAFLD, Mayo Clinic
- Weight loss → DGA, Mediterranean, ACSM
- Muscle gain → ISSN Sports Nutrition, ACSM
- Healthy eating / prevention → DGA, WHO, Mediterranean
- Anything else → Mayo Clinic + EFSA + WHO + USDA

The skill uses WebFetch to pull current text from those sources (or cached from `references/guidelines_cache.md` if <90 days old) and proposes Primary / Secondary / Contextual parameters + variable modifiers + quality adjustments. You confirm or refine. The derived setup is saved to your local `profile.md`.

---

## Cuisine knowledge base

`cuisines/*.md` files contain condition-agnostic structural facts per cuisine: standard dishes, component breakdown, typical macros, cooking method, what's modifiable, common variants, confidence.

Seed includes 6 cuisines: **Italian, Japanese, Thai, Mediterranean, American, French**.

Add a new cuisine by copying `cuisines/_template.md` and filling it in. The skill auto-reads any new `cuisines/*.md` you add.

---

## License

MIT — use freely, modify freely, share freely.
