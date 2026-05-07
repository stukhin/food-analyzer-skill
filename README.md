# 🥗 Food Analyzer Skill for Claude

A Claude skill that analyzes food photos, menu screenshots, and dish descriptions to evaluate nutritional content and health suitability — personalized to your dietary goals or medical condition. Works in English and Russian (replies in whichever language you start the conversation in).

---

## What it does

- **Analyzes food photos** — identifies ingredients, estimates КБЖУ / macros as ranges with confidence indicator
- **Reads menu screenshots** — multi-dish comparison table with visual fat bars
- **Scores each dish** on a 1–10 scale with a green/yellow/red stoplight, looked up in deterministic threshold tables
- **Adapts to your condition** — gallbladder, gastritis, diabetes, hypertension, dyslipidemia, weight loss, muscle gain, general healthy eating
- **Tags every health claim** with an evidence level (🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate) — no fake authority
- **Tracks daily budget** (opt-in) — per-day macro budget with verdicts that shift in context
- **Persists your profile** — onboarding runs once; reads back on every future session (Claude Code)
- **Accepts corrections** — tell it what it got wrong, it recalculates honestly

---

## Key principle

This skill is intentionally critical. It will not soften bad scores to make you feel better.

- **Red is red.** No "best of bad options" in STRICT mode (acute / post-op / diagnostic).
- In PRAGMATIC mode (remission / general life), ranking is allowed — but if the top option is yellow or worse, the skill explicitly says so and describes what a truly green choice would look like.
- Scoring is deterministic: every threshold table uses non-overlapping value bands mapped to single integer scores. No vibes.

---

## Install

### Claude Code (personal skill)

```bash
git clone https://github.com/stukhin/food-analyzer-skill.git ~/.claude/skills/food-analyzer
```

Restart Claude Code (or start a new session). The skill activates automatically when you describe food or send a photo.

### Claude.ai (manual)

Copy `SKILL.md` and paste it at the start of a conversation as a system instruction, then send your photo or dish description.

---

## How to use

**First session**: Claude asks 3 onboarding questions (condition, foods to avoid, current phase: acute or remission). Profile is saved to `~/.claude/skills/food-analyzer/profile.md`.

**Subsequent sessions**: Profile loads automatically. Skill confirms it once and skips onboarding.

**Inputs**:
- Send a photo of food → table-format breakdown + verdict
- Send a menu screenshot → comparison table with fat bars + ranking
- Describe a dish in text → estimated analysis
- Say *"веду дневник"* / *"track my day"* → activates Daily Tracker mode

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
- Cooking method (steamed / grilled / pan-fried / deep-fried) as a modifier
- Overall weighted via Primary / Secondary / Contextual hierarchy

---

## Supported conditions

| Goal / condition | Primary concerns |
|-----------|-----------------|
| Gallbladder / biliary sludge | Total fat per meal, saturated fat |
| Gastritis / peptic ulcer | Acidity, spice |
| Type 2 diabetes | Carbs, added sugar |
| Hypertension | Sodium |
| Dyslipidemia / high LDL | Saturated fat, trans fat |
| Weight loss | Calories, fat density |
| Muscle gain | Protein |
| General healthy eating | Saturated fat, added sugar, sodium |

If your condition isn't listed (e.g. IBS, FODMAP, kidney CKD, gout, GERD), the skill derives 3–5 priority parameters from the relevant guideline (Mayo Clinic, EFSA, WHO) and tells you upfront which ones it will weigh heaviest.

---

## Data sources

- [EASL Clinical Practice Guidelines on Gallstones (2016)](https://easl.eu)
- [AHA Dietary Guidelines](https://www.heart.org)
- [ADA Standards of Medical Care in Diabetes](https://diabetesjournals.org/care)
- [ESC/EAS Dyslipidaemia Guidelines](https://www.escardio.org)
- [DASH / NHLBI hypertension guidelines](https://www.nhlbi.nih.gov)
- [ACG (American College of Gastroenterology)](https://gi.org)
- [WHO daily intake reference](https://www.who.int)
- [USDA FoodData Central](https://fdc.nal.usda.gov)
- [Mayo Clinic patient resources](https://www.mayoclinic.org)
- Pevzner Diets №1 / №5 / №9 — Russian dietary tradition (marked explicitly)
- DGA (US Dietary Guidelines for Americans) + Mediterranean diet pattern

---

## Evolving this skill

### Add a new condition

In `SKILL.md`:
1. Add a threshold table block with non-overlapping value bands → single integer scores.
2. Add a row to the **Adaptive Scoring (Hierarchy)** table with Primary / Secondary / Contextual parameters.
3. Update the README condition table.

### Change the output format

The single-dish table format and bar specs live in the **Visual Output** and **Single Dish Output Format** sections of `SKILL.md`.

### Prompt for Claude Code to evolve this skill

```
I have the food-analyzer skill at ~/.claude/skills/food-analyzer/SKILL.md.

Here's what I want to change:
[describe]

Please:
1. Read the current SKILL.md.
2. Propose the specific edits as a diff.
3. Wait for confirmation before applying.
4. Suggest 2-3 test cases to verify the change.
```

---

## Contributing

Pull requests welcome. If you add support for a new medical condition or dietary protocol:

1. Add the threshold table to `SKILL.md` with cited sources.
2. Add the condition to the hierarchy table in `SKILL.md`.
3. Add the condition to the README condition table.
4. Include at least 2 example dishes with expected scores in the PR description.

---

## License

MIT — use freely, modify freely, share freely.
