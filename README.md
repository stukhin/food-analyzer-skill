# 🥗 Food Analyzer Skill for Claude

A Claude skill that scores food photos, menus, and dish descriptions against **your personal medical condition or dietary goal** — using current published guidelines as the evidence base, not vibes.

> *"I tested it on a Hawaiian Loco Moco, expecting it to call it bad. It found four independent fat sources I hadn't noticed (mac salad is mayonnaise on macaroni, the gravy is built on rendered beef fat, the wonton crisps are deep-fried, the patty is pan-fried), scored it 1/10 🔴 with the actual per-meal target cited from EASL, and showed me a modified version that would land at 6/10 🟡. I switched my order."*

**Bilingual.** The skill runs entire sessions in either **English** or **Russian** — the user picks at onboarding (Step 0.5). Dish/menu names in any language are preserved verbatim; only the analysis prose is localized.

Single-user skill (one install = one profile). Public, MIT-licensed.

[Install](#installation) · [Live examples](#-live-examples-text-based) · [Architecture](#-architecture) · [How scoring works](#how-scoring-works) · [Supported conditions](#supported-conditions) · [Bilingual support](#bilingual-support)

---

## What's new in v3.2.0

- **Explicit language selection at onboarding (Step 0.5).** The skill now *asks* — English or Russian? — instead of auto-detecting from the first message. The choice is stored in `profile.md` as `communication_language` and used for every subsequent output.
- **Foreign-content preservation rule** — dish and menu names in any language (Chinese, Japanese, Thai, Korean, Filipino, etc.) stay verbatim in the output. Only labels, prose, verdicts, modifications, and footers are localized.
- **Methodology document (`SKILL.md`) is now fully in English.** Templates show English labels as the default; a complete **Localization table** maps every label to its Russian equivalent. The model substitutes at runtime based on the user's choice.
- **Honesty Rule 14 — Language compliance is mandatory.** Mid-message language drift is a violation. Switching languages only happens on explicit user request.
- **Anti-Patterns #14 and #15** — don't translate dish names; don't mid-message-drift.

Everything else (scoring methodology, knowledge-base architecture, 6 cuisines, indulgence framework, evaluation heuristics, format compliance from v3.1) carries forward unchanged.

Full version history in [CHANGELOG.md](CHANGELOG.md).

---

## What it does (one paragraph)

You install the skill. At first use, it runs a six-step wizard that confirms the language, captures your basics, diagnosed conditions, current phase (acute/remission/preventive), goals, restrictions (medical/religious/ethical/personal), and personal calibration. It looks up current guidelines for your condition(s) via WebFetch (EASL for gallstones, ADA for diabetes, ACG for gastritis, DASH for hypertension, ESC/EAS for dyslipidemia, KDIGO for kidney, ACR for gout, AASLD for fatty liver, Monash for IBS, DGA for general healthy eating — full source map below). It proposes a personalized threshold setup with non-overlapping value bands. You confirm or refine. From then on, every food photo or menu you send gets evaluated against your derived profile, producing a deterministic 1–10 score per parameter with stoplight color, a verdict, modification suggestions with explicit re-scoring delta, and a sensitivity disclosure.

The skill never claims medical authority. Every health-related claim is tagged with an evidence level — 🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate — and on red verdicts it appends a one-line disclaimer.

---

## 🎬 Live examples (text-based)

These are actual outputs the skill produces, copied as Claude would render them in your chat. No screenshots — text only, so you can read every detail. All examples use English as the chosen `communication_language`; one bilingual example at the end shows the Russian variant.

### 1. First-time onboarding (compressed walkthrough)

> **User:** rate this dish for me [photo attached]
>
> **Skill:** I'm a food-analysis tool, not medical advice. My scoring is based on published dietary guidelines — it doesn't replace your physician or dietitian. For serious conditions, consult a specialist.
>
> Before we start: **which language should I use for our conversation? English or Russian?** Menus and dish names in any language are fine — I'll keep them as-is and respond in your chosen language for everything else.
>
> **User:** English
>
> **Skill:** English it is. Quick onboarding before I score anything — five short groups, one at a time.
>
> **Group 1 — Basics**
> - Age range? (e.g. 25–34, 35–44, 45–54)
> - Sex (M / F / other)?
> - Height / weight — optional, for BMI context
> - Activity level (sedentary / light / moderate / active / very active)?
>
> **User:** 35–44, M, 178cm/82kg, light
>
> **Skill:** Got it.
>
> **Group 2 — Health context**
> - Diagnosed conditions? (diabetes T1/T2, GERD, IBS/IBD, celiac, hypertension, high cholesterol, MASLD/fatty liver, gallstones, food allergies, kidney disease, autoimmune, history of disordered eating, etc.)
> - Medications / supplements that affect digestion?
> - Current phase — *acute / postoperative / diagnostic* (→ STRICT mode), or *stable remission / general prevention* (→ PRAGMATIC mode, default)?
>
> **User:** gallstones (4 stones, 0–5mm, asymptomatic, found incidentally), slightly tortuous common bile duct, chronic gastritis but no symptoms. Stable. No meds.
>
> **Skill:** Two conditions — I need the priority before researching: which one drives my primary parameters right now? Gallstones with tortuous CBD sounds structurally riskier; gastritis is currently silent. Confirm or correct.
>
> **User:** gallbladder priority
>
> **Skill:** Confirmed. Continuing.
>
> **Group 3 — Goals**
> - Weight target — lose / maintain / gain?
> - Preferred dietary pattern if any (Mediterranean, DASH, low-FODMAP, keto, etc.)?
> - Tracking preference — single meal (per-dish in isolation) or daily (running budget)?
>
> **User:** maintain, no specific pattern, single meal
>
> **Group 4 — Restrictions**
> - Medical / allergies?
> - Dietary pattern (vegan / vegetarian / pescatarian / halal / kosher / none)?
> - Personal aversions?
> - Cuisines you eat often?
> - Eating schedule (meals/day, intermittent fasting)?
>
> **User:** no allergies, no pattern, dislike cilantro, eat Filipino + Japanese + American often, 3 meals
>
> **Group 5 — Calibration**
> - Foods that historically gave you symptoms (gallbladder pain, gastritis flare, anything)?
> - Foods that consistently feel safe?
> - Personal aids (enzymes, supplements taken with meals)?
> - Anything in your history that should change how I report numbers (history of eating disorder, obsessive macro tracking, restrictive escalation)?
>
> **User:** rich cream sauces gave dull right-side ache once; grilled fish + rice always feels good. No ED history. No supplements.
>
> **Skill:** Logged. Now researching your priority condition. *(WebFetch: EASL Gallstone Guidelines 2016; cross-check with Stol №5; ACG gastritis for secondary)*
>
> *(40 seconds later)*
>
> Profile proposal for **gallbladder priority (gallstones + tortuous CBD) + silent chronic gastritis**, PRAGMATIC mode:
>
> Sources used:
> - **EASL Clinical Practice Guidelines on Gallstones, 2016** [🔵 Guideline] — fetched today, https://easl.eu/...
> - **ACG Gastritis Guideline** [🔵 Guideline] — fetched today, https://gi.org/...
> - **Stol №5** [🔵 Guideline, marked as Russian dietary tradition] — for the per-meal fat-distribution principle
>
> **Note on a source conflict:** EASL does NOT require strict fat restriction for asymptomatic gallstones (recommends moderate fat, omega-3 beneficial). Stol №5 is stricter (70–80g/day total fat, distributed dribble-style across 4–5 meals). Given your tortuous CBD + actual stones (not just sludge), I'm taking a pragmatic middle — closer to Stol №5 on fat *type* and cooking method, looser on absolute grams since you're asymptomatic. Surfacing both so you know.
>
> **Primary parameters** (any one in red → whole verdict 🔴):
>
> *Fat per meal* (target ~15–20g):
> ```
> ≤15g     → 10  🟢
> >15 ≤25g → 8   🟢
> >25 ≤35g → 5   🟡
> >35 ≤50g → 3   🔴
> >50g     → 1   🔴
> ```
> *Saturated fat per meal:*
> ```
> ≤5g      → 10  🟢
> >5 ≤8g   → 7   🟡
> >8 ≤12g  → 4   🟡
> >12g     → 1   🔴
> ```
> *Cooking method* (modifier on fat score):
> ```
> steam/boil/poach/raw    +0
> grill/bake              +0
> light sauté/braise      −1
> pan-fry                 −2
> deep-fry                −3
> ```
>
> **Secondary parameters** (any one red with green Primary → 🟡):
> - Acidity / spice / coffee / alcohol — gastritis triggers, even though silent
> - Fiber (≥5g/meal is good; soluble preferred)
>
> **Variable-size modifiers:**
> - Omega-3 bonus +1 when fatty-fish portion present
> - Fat-shock penalty −2 when concentrated saturated fat at one sitting (cream, butter, lard, foie gras)
> - Spicy-trigger penalty −1 when overtly spicy on empty stomach
>
> **Quality adjustments:**
> - Olive / avocado oil — neutral
> - Cream / butter / lard — −1 (animal saturated)
> - Coconut milk / cream — −1 (plant but high-saturated)
> - Omega-3 fish (salmon, sardines, mackerel) — +1
>
> Confirm, adjust, or call out something I missed.
>
> **User:** Confirmed.
>
> **Skill:** Profile saved to `~/.claude/skills/food-analyzer/profile.md`. Backup at `backups/profile.md.backup-20260527-153012`. Send a dish to start.

This whole conversation runs once. Subsequent sessions skip the wizard and just confirm the loaded profile in one line.

---

### 2. Mode A — Single dish (Hawaiian Loco Moco)

The actual format the skill is required to produce for every single-dish evaluation:

```
**Dish:** Loco Moco (Hawaiian comfort plate)
**Confidence:** macros M · cooking H · portion L · sauce/composition L

Beef patty (pan-fried) + white rice + mushroom gravy + poached egg +
mac salad (macaroni in mayonnaise) + wonton crisps (deep-fried).
Heuristic triggers: #1 (healthy-sounding trap doesn't apply — but
"mac salad" sounds innocuous when it's mayonnaise on macaroni) and #2
(hidden fat — gravy built on rendered beef fat + deep-fried wontons).

**Macros (range):**
- Calories: 950–1180 kcal
- Protein:  38–48g
- Fat:      45–65g (saturated 16–24g)
- Carbs:    90–110g

**Scoring:**

| Parameter   | Value              | Score | Bar         |
|-------------|--------------------|-------|-------------|
| Fat/meal    | 45–65g             | 1/10  | █░░░░░░░░░ 🔴 |
| Sat fat     | 16–24g             | 1/10  | █░░░░░░░░░ 🔴 |
| Cooking     | pan-fry + deep-fry | −3    | modifier     |
| **Total**   | —                  | 1/10  | █░░░░░░░░░ 🔴 |

**Verdict:** skip — four independent fat sources in one sitting,
2.5× over per-meal target [🔵 EASL].

**Why this score:** four independent fat sources stack (patty, mac-salad
mayo, gravy, deep-fried wontons), plus a double-cooking-method hit
(pan-fry + deep-fry). Any one would push yellow; together they trip
the first Primary red.

**Modifications:**
- Plain steamed rice instead of garlic fried rice → −12g fat.
- Remove wonton crisps → −10g fat.
- Grilled chicken breast instead of patty + gravy on the side → −20g fat.

**Re-scored with modifications:** as served 1/10 🔴 → modified (grilled
chicken + gravy on side + plain rice + no crisps) 6/10 🟡.

**Most uncertain:** gravy volume and mac-salad dressing quantity
(could add ±10g fat).

⚠️ Not medical advice. Acute symptoms — see a physician.

*Evaluating in isolation — tell me what you've eaten today if you want a daily balance.*

*Tell me if I misread anything.*
```

What this format guarantees: confidence breakdown by 4 aspects, macro ranges (not single numbers), a scored parameter table with bars, single-sentence verdict, reasoning inside `**Why this score:**`, modifications with a concrete re-scored delta line, "most uncertain" call-out, mandatory disclaimer on red, dual footers.

---

### 3. Mode A — Healthy-sounding trap detection (Wellness Bowl)

User profile: T2 diabetes, stable, PRAGMATIC, `communication_language: en`.

```
**Dish:** "Wellness Bowl" (label) / poke-style bowl with salmon
**Confidence:** macros M · cooking M · portion M · sauce/composition L

🚩 Healthy-sounding trap detected. The "wellness" label does not lower
the estimate: visible composition shows tempura crisps on top, "spicy mayo"
drizzle, imitation crab with mayonnaise, avocado, edamame, rice. Heuristic
#1 (trap detection) and #2 (hidden fat in two mayo-based sauces).

**Hidden fat sources:**
- Spicy mayo — ~2 tbsp ≈ 12–18g fat (mostly saturated + processed)
- Imitation crab salad — typically with mayo, ~5–8g fat
- Tempura crisps — deep-fried, ~5–10g fat absorbed

**Macros (range):**
- Calories: 620–820 kcal
- Protein:  24–32g
- Fat:      28–42g (saturated 6–9g)
- Carbs:    65–80g (sugar 8–12g)

**Scoring:**

| Parameter      | Value      | Score | Bar         |
|----------------|------------|-------|-------------|
| Carbs          | 65–80g     | 4/10  | ████░░░░░░ 🟡 |
| Added sugar    | 8–12g      | 6/10  | ██████░░░░ 🟡 |
| Fiber          | 5–8g       | 8/10  | ████████░░ 🟢 |
| Sat fat        | 6–9g       | 7/10  | ███████░░░ 🟢 |
| Omega-3        | salmon     | +1    | bonus        |
| **Total**      | —          | 5/10  | █████░░░░░ 🟡 |

**Verdict:** caution — high total carb load plus hidden mayo-based fats
push it into yellow despite the strong salmon-and-fiber base [🔵 ADA: ≤45g
carbs/meal target for T2].

**Why this score:** salmon + edamame + avocado is a strong foundation,
but two mayo-based sauces and tempura crisps add 25g+ fat that doesn't
appear on the label. Carbs at 65–80g are also above ADA per-meal range.
The omega-3 bonus and good fiber save it from red.

**Modifications:**
- "Spicy mayo on the side" → −12–18g fat, color → 🟢
- "Hold the tempura crisps" → −5–10g fat
- "Imitation crab replaced with extra edamame" → −5–8g fat, +protein quality

**Re-scored with modifications:** as served 5/10 🟡 → modified
(sauce on side + no tempura + edamame for imitation crab) 8/10 🟢.

**Most uncertain:** actual mayo volume. If drizzle = 1 tsp vs 2 tbsp,
the whole dish swings 🟡 → 🟢 without any modifications.

**Lesson:** "wellness / bowl / protein" in a name is not evidence.
Check sauces and toppings.

*Evaluating in isolation — tell me what you've eaten today if you want a daily balance.*

*Tell me if I misread anything.*
```

---

### 4. Mode A — Correction flow with re-scoring

After the Wellness Bowl was scored, the user says:

> **User:** actually it was half a bowl, the rest got boxed for tomorrow

The skill recalculates:

```
Got it — recalculating.

**Macros (recomputed for half portion):**
- Calories: 310–410 kcal
- Fat:      14–21g (saturated 3–5g)
- Carbs:    32–40g

**Scoring:**

| Parameter   | Value     | Score | Bar         |
|-------------|-----------|-------|-------------|
| Carbs       | 32–40g    | 8/10  | ████████░░ 🟢 |
| Sat fat     | 3–5g      | 9/10  | █████████░ 🟢 |
| Fiber       | 3–4g      | 7/10  | ███████░░░ 🟢 |
| **Total**   | —         | 8/10  | ████████░░ 🟢 |

**Verdict:** good choice in the new portion — half the bowl puts you
in ADA per-meal targets on all axes.

**Why this score:** portion size was the only thing that changed.
Composition (sauces, tempura) is the same, but absolute grams halved
and parameters moved into green. Logging as a successful behavioral
modifier for this format.

Logged to calibration log: *2026-05-27 — wellness bowl half portion —
ok, no symptoms expected based on composition*.

*Tell me if I misread anything.*
```

Note what didn't change: the skill did NOT lower the per-100g composition score (per Honesty Rule 4). It re-estimated the serving size, which legitimately moves absolute macros into target range.

---

### 5. Mode B — Multi-dish ranking (PRAGMATIC mode)

User sends a screenshot of five dishes from a Macro Bowls menu. Profile: gallbladder priority, PRAGMATIC, `communication_language: en`.

Note: dish names are preserved verbatim from the menu (English here, but they'd stay the same in a Russian-language session — only the surrounding analysis would localize).

```
Five dishes from one menu — all on white rice + the same steamed vegetables,
so the difference is in protein and sauce.

| # | Dish                  | Price | Fat (10-cell bar)   | Score    |
|---|-----------------------|------:|---------------------|---------:|
| 1 | Chicken Oyakudon      | ₱280  | █░░░░░░░░░ 12–18g   | 8/10 🟢   |
| 2 | Eggs in Tomato        | ₱260  | ██░░░░░░░░ 15–22g   | 6/10 🟡   |
| 3 | Ham Omelette          | ₱290  | ████░░░░░░ 25–32g   | 5/10 🟡   |
| 4 | Chicken Salpicao      | ₱320  | █████░░░░░ 30–38g   | 4/10 🟡   |
| 5 | Chicken Ala King      | ₱350  | ████████░░ 40–55g   | 2/10 🔴   |

## Top picks — why
- **Chicken Oyakudon (8/10 🟢)** — grilled chicken breast + lightly cooked
  egg in soy-mirin sauce. Cooking method grilled (+0); sauce not built on
  fat. Only flag is sugar/salt in the teriyaki for gastritis — but yours
  is silent. The cleanest pick for the gallbladder profile.

## Worst — why avoid
- **Chicken Ala King (2/10 🔴)** — description gives it away immediately:
  *"rich, creamy sauce made with cream, butter."* Cream + butter = direct
  fat-shock penalty (−2), saturated fat in the red zone (>12g per
  serving). A classic "healthy macro bowl" trap: grilled chicken and
  steamed vegetables mask a cream-and-butter base. Fat 40–55g, red on the
  first Primary.
- **Chicken Salpicao (4/10 🟡)** — salpicao is traditionally cooked in
  garlic-butter, with a fried egg on top. The breast itself is lean,
  but the sauce and the egg together pull fat to 30–38g.

## What to order
**Chicken Oyakudon, no modifications.** Reasonable price, clean green for
the profile. If you want eggs — Eggs in Tomato, ask for less oil and
account for tomato acidity (tolerable in gastritis remission). Skip
Chicken Ala King; "grilled chicken + steamed veg" in the description
doesn't cancel the cream sauce.

*Confidence: medium across the board. I don't see sauce gram amounts,
but the compositions and typical recipes for these dishes are
unambiguous.*

*Want a detailed breakdown for any of these?*
```

---

### 6. Mode B — STRICT mode (acute flare)

Same five dishes, but the user is now in an acute gallbladder flare. STRICT mode kicks in.

```
**STRICT mode — not ranking the options, because they're all incompatible
with an acute flare.**

All five dishes contain either grilled meat with sauces (Oyakudon,
Salpicao, Ala King, Ham Omelette) or fried egg (Eggs in Tomato). In an
acute phase with gastritis + gallbladder, all are 🟡/🔴 territory.

**Don't eat here right now.** A suitable venue during your flare would offer:
- Clear-broth soups (miso, dashi, chicken consommé, lean vegetable soup)
- Steamed white fish (cod, sole, snapper, hamachi sashimi if you can do raw)
- Plain steamed rice + boiled chicken breast (white meat only)
- Soft-boiled or poached eggs (one yolk maximum per meal)
- Steamed / boiled vegetables without oil
- Avoid: anything fried, any cream/butter sauces, any tomato/citrus/vinegar,
  any spicy seasoning, coffee, alcohol.

If you can't find a suitable venue, eat at home today. Once you're back
to remission, this menu has workable options — I'll re-rank then.

*Tell me if I misread anything, or if your phase has updated and you're
already in remission.*
```

In STRICT, the indulgence framework is disabled (Honesty Rule 11). No "least bad" ranking.

---

### 7. Indulgence framework — treats in PRAGMATIC

User profile: dyslipidemia, stable, PRAGMATIC, `communication_language: en`. They ask:

> **User:** Can I have ice cream once a week?

```
**Can you have this?** — yes, but as risk management, not "just because".
Premium full-fat ice cream (not sorbet) is problematic for your profile
because of concentrated saturated fat (heavy cream + egg yolks).

**What's specifically problematic here:**
- 1 scoop premium ice cream ≈ 8–14g fat, of which 5–9g saturated.
  ESC/EAS per-meal target <7g saturated [🔵 ESC/EAS].
- Sugar 12–22g/scoop — sweet-drink-like spike.

**Stacking risk modulators:**
- **One scoop, not two or three** — the biggest single lever, moves 🔴 to 🟡.
- **After a light meal**, not after pasta carbonara.
- **Not on an empty stomach** — needs food in the system.
- **Earlier in the day** — after lunch better than after dinner.
- **15-minute walk after** — supports digestion and glucose.

**Red combinations to avoid:**
- Large portion (2–3 scoops) + after fatty dinner + late evening — three "bads" stacked.
- Ice cream + sweet cocktail — double sugar load.

**Lighter alternatives (same niche):**
- **Sorbet** (fruit ice) — 0g saturated fat, similar sugar, lands at 🟡.
- **Dark chocolate gelato** small scoop — saturated fat ~3g vs 8g.
- **Frozen yogurt** — but **wellness-label trap, check composition**:
  "low fat" version can be ok, "creamy" version = same as ice cream.

**Frequency:** once every 1–2 weeks with the modulators above — clinically
acceptable for dyslipidemia in remission [⚪ Clinical]. More frequent —
cumulative effect on LDL.

⚠️ Not medical advice.

*Tell me if I misread anything.*
```

---

### 8. Daily Tracker mode

Activated when the user mentions tracking the day. Profile: gallbladder.

> **User:** tracking today — for breakfast I had a 2-egg omelet + toast + black coffee

```
**Daily Tracker on.** Limits I'm using for your profile:
- Calories: 1800 kcal
- Fat: 75g, distributed across 4–5 meals (~15–20g per meal, Stol №5
  daily target)
- Sat fat: 20g
- Sodium: 2000mg

**Breakfast logged:**
- 2-egg omelet — ~14g fat (10g sat), 16g protein, 1g carb
- Toast — 80 kcal, 1g fat, 15g carb
- Black coffee — 0 macros
- Breakfast subtotal: ~280 kcal, 15g fat, 17g protein, 16g carb

**Daily balance so far:**
```
  Calories:  280 / 1800   (16%)   remaining 1520
  Fat:       15g / 75g    (20%)   remaining 60g
  Sat fat:   10g / 20g    (50%)   ⚠️ remaining 10g
  Sodium:    180 / 2000   (9%)    remaining 1820
```

**Note on saturated fat:** you're already at 50% of the daily limit
after breakfast (yolks + butter in the omelet). Keep lunch and dinner
on lean protein (fish, lean chicken breast, legumes) — otherwise you'll
go over by evening.

*Send the next dish — I'll score it in the daily-balance context.*
```

After the next meal, the skill applies both per-meal scoring AND the budget layer:

```
**Dish:** Salmon Teriyaki Bowl
[... standard Mode A output with table ...]

**Per-meal verdict:** 7/10 🟢

**In today's context:**
- +14g fat → daily total 29g/75g (39%) ✓
- +3g sat fat → daily total 13g/20g (65%) ✓ (still fits)
- Omega-3 bonus from salmon ✓ — the day is balancing well.

Verdict holds at 🟢 in today's context.

**Daily balance after lunch:**
[... updated budget table ...]
```

If saturated fat had pushed over the daily limit, the budget layer would shift verdict color independently of per-meal — the only mechanism in v3.1+ allowed to cross color outside the per-meal lookup (Layer Separation rule).

---

### 9. Calibration log

After a Mode A evaluation, the user volunteers:

> **User:** yesterday's pork ribs gave me a right-side ache for about three hours, uncomfortable

The skill responds briefly:

```
Logged.

→ `~/.claude/skills/food-analyzer/profile.md > Personal calibration log`:
   - 2026-05-27 — fried pork ribs — right-side dull ache, ~3 hours

I won't change thresholds from one observation (too noisy). If a similar
reaction repeats two more times on similar fatty dishes, I'll surface the
pattern and propose tightening the fat-shock modifier.
```

After three consistent entries on similar triggers, the skill surfaces the pattern:

```
Over the past month, three dishes with >35g fat at a single sitting gave
you right-side ache lasting 2–4 hours. This is your personal fat-shock
signature. Proposal:
- tighten per-meal fat cap from 35g → 30g (Primary band shifts)
- strengthen fat-shock penalty from −2 to −3

Apply? I'll back up the profile, rebuild, and show the diff. Won't touch
anything without your "yes".
```

The user controls all profile changes. The skill suggests, the user confirms.

---

### 10. Mode C — Meal planning (v3.0 minimal skeleton, full mechanics in later release)

> **User:** what should I eat today

```
**Plan for today** (gallbladder priority + silent gastritis, PRAGMATIC,
~70–80g daily fat split across 4–5 meals).

### Breakfast
- Oatmeal cooked in water + 1 banana + 1 tbsp almond butter + black coffee
- ~350 kcal, 10g fat (mostly mono-unsaturated), 50g carb, 9g protein
- Score: 8/10 🟢

### Lunch
- Grilled salmon (150g) + brown rice (100g cooked) + steamed broccoli
- ~520 kcal, 18g fat (omega-3 dominant), 35g carb, 35g protein
- Score: 9/10 🟢, omega-3 bonus stacking

### Snack
- Apple + 6 raw almonds
- ~140 kcal, 6g fat, 22g carb, 3g protein

### Dinner
- Chicken breast (120g) + quinoa (80g cooked) + roasted zucchini + tomato
- ~420 kcal, 8g fat, 38g carb, 32g protein
- Score: 9/10 🟢

**Daily totals (computed):**
- Calories: ~1430 (maintenance ok)
- Fat: ~42g (well under 75g cap, distributed across 4 sittings —
  Stol №5 dribble principle respected)
- Sat fat: ~7g (well under 20g cap)
- Omega-3 portion present at lunch ✓

**Variations and rotation:** salmon at lunch is the primary omega-3 hit
for the day. If you repeated salmon yesterday, swap for cod / sea bass +
add walnut to the snack for omega-3. Quinoa at dinner is one option;
brown rice works too. Cuisine rotation gentle, not three uniform days
in a row.

*Full Mode C mechanics (variety enforcement, shopping list, multi-day
running totals) — coming in a later release. This is the minimal v3.0
skeleton.*
```

---

## Bilingual support

The skill runs entire sessions in either **English** or **Russian** based on `communication_language` from the profile. The choice is asked explicitly at onboarding Step 0.5.

### What gets localized

| Element | Localized? |
|---|---|
| Section labels in Mode A/B templates (`Dish`, `Confidence`, `Macros`, etc.) | YES |
| Verdict prose, modifications, "why this score" | YES |
| Footers, disclaimer | YES |
| Dish / menu names | **NO — preserved verbatim** |
| Source citations (EASL, ADA, USDA, Stol №5) | **NO — canonical** |
| Evidence icons (🔵 🟣 ⚪ ⚫), stoplight (🟢 🟡 🔴), score bars (█████░░░░░) | **NO — universal** |
| Numbers, units (g, kcal, mg) | **NO — universal** |

### Example — same dish in Russian

The same Loco Moco evaluation from Example 2, with `communication_language: ru`:

```
**Блюдо:** Loco Moco (Hawaiian comfort plate)
**Уверенность:** макро M · готовка H · порция L · соус/состав L

Говяжья котлета (пан-фрай) + белый рис + грибная грэви + пашот-яйцо
+ mac salad (макароны в майонезе) + вонтон-крисы (фритюр).
Триггеры heuristic'ов: #1 (mac salad — это майонез на макаронах,
не салат), #2 (грэви на жире, вонтоны фритюр).

**Макро (диапазон):**
- Калории: 950–1180 kcal
- Белок:   38–48g
- Жир:     45–65g (насыщ. 16–24g)
- Углеводы: 90–110g

**Оценка:**

| Параметр    | Значение           | Балл | Бар         |
|-------------|--------------------|------|-------------|
| Жир/приём   | 45–65g             | 1/10 | █░░░░░░░░░ 🔴 |
| Насыщ. жир  | 16–24g             | 1/10 | █░░░░░░░░░ 🔴 |
| Готовка     | пан-фрай + фритюр  | −3   | модификатор   |
| **Итого**   | —                  | 1/10 | █░░░░░░░░░ 🔴 |

**Вердикт:** пропустить — четыре независимых жировых источника
в одном приёме, 2.5× превышение per-meal target [🔵 EASL].

**Почему этот балл:** стекаются четыре независимых жировых источника,
плюс двойной удар по готовке (пан-фрай + фритюр).

**Что можно изменить:**
- Plain steamed rice вместо garlic fried rice → −12g жира.
- Убрать вонтон-крисы → −10g жира.
- Grilled chicken breast вместо котлеты + грэви на сторону → −20g жира.

**Пересчёт с модификациями:** as served 1/10 🔴 → modified 6/10 🟡.

**Самое неопределённое:** объём грэви и заправки в mac salad
(могут добавить ±10g жира).

⚠️ Не медицинская рекомендация. При острых симптомах — к врачу.

*Оцениваю в изоляции — скажи, что ел до этого, если хочешь дневной баланс.*

*Скажи, если что-то распознал не так.*
```

Note: "Loco Moco", "mac salad", "garlic fried rice", "grilled chicken breast" — all preserved verbatim. Only labels and reasoning prose are in Russian.

### Switching languages mid-session

The session language is sticky once set. To switch, say *"switch to Russian"* / *"переключи на английский"*. The skill backs up the profile, updates `communication_language`, and continues in the new language. Random mid-message language drift by the user does NOT switch the session.

---

## 🏛 Architecture

The skill is no longer a single file. It's a structured knowledge base under `~/.claude/skills/food-analyzer/`:

```
food-analyzer/
├── SKILL.md                    ← methodology (1450+ lines, English-only)
│                                  • workflows, output templates, 14 honesty rules,
│                                    15 anti-patterns, 5 heuristics, indulgence
│                                    framework, drinks hierarchy, etc.
│                                  • ZERO personal data, ZERO condition-specific
│                                    numbers
│                                  • Localization table maps English labels to
│                                    Russian equivalents at runtime
│
├── profile.md                  ← YOUR personal data + derived thresholds
│                                  • communication_language (en | ru)
│                                  • condition(s), phase, restrictions,
│                                    personal aids, calibration log
│                                  • generated by the wizard, edited only via
│                                    skill's update flow
│                                  • GITIGNORED — never leaves your machine
│
├── cuisines/                   ← reusable, condition-agnostic
│   ├── _template.md            ← copy this to add a new cuisine
│   ├── italian.md              ← 14 dishes (pasta light/heavy fork)
│   ├── japanese.md             ← 16 dishes (broth fork, mazesoba oil pool)
│   ├── thai.md                 ← 13 dishes (tom yum vs tom kha)
│   ├── mediterranean.md        ← 12 dishes (olive oil quality)
│   ├── american.md             ← 14 dishes (combo vs à la carte)
│   └── french.md               ← 14 dishes (sauce-on-side culture)
│
├── references/
│   ├── _sources.md             ← registry of 19 authoritative bodies
│   │                              EASL, AASLD, ACG, ADA, AHA, ESC/EAS,
│   │                              KDIGO, NKF, ACR, EULAR, NICE, Monash,
│   │                              Pevzner, DGA, WHO, EFSA, USDA, Mayo, ISSN
│   └── guidelines_cache.md     ← cached condition summaries, 90-day refresh
│
└── backups/                    ← auto-snapshots of profile.md before any edit
                                   last 5 retained, GITIGNORED
```

**Key separation:** `SKILL.md` never holds your data; `profile.md` never holds methodology. Updating either one doesn't touch the other. Cuisines/references can be extended without touching either.

---

## How scoring works

Five things define the score for any dish:

1. **Primary parameters** (from your profile). The verdict-driving axes — usually 2–3 of them — e.g. for gallbladder it's fat-per-meal + saturated-fat + cooking method. **Any Primary in red (1–3) → whole verdict 🔴**, regardless of others.

2. **Secondary parameters** (from your profile). Important but not verdict-driving alone. Any Secondary in red with all Primary green → 🟡.

3. **Contextual modifiers** (from your profile). Cooking method nudges, processing level, temperature. Shift the numeric score ±1 but **never cross color boundaries** at the per-meal layer.

4. **Variable-size modifiers** (from your profile). Named bonuses and penalties with declared sizes — e.g. omega-3 bonus +1 when fatty-fish present, fat-shock penalty −2 when concentrated saturated fat. These CAN cross color boundaries in either direction (with justification).

5. **Quality adjustments** (from your profile). Type matters as much as quantity. 25g of olive oil ≠ 25g of cream/lard at the same gram count. The profile declares the rule; the engine applies it.

**Verdict cascade:**

```
Any Primary in 1–3 (red)?            →  🔴 red overall (override)
                ↓ no
All Primary 8–10 (green) AND
all Secondary 8–10 (green)?          →  🟢 green
                ↓ no
Any Secondary in 1–3 with all
Primary 8–10?                        →  🟡 yellow
                ↓ no
Mixed Primary (some 8–10, some
4–7)?                                →  🟡 yellow
```

Bands per parameter are **non-overlapping value-bands** mapped to **single integer scores** — no overlap, no ambiguity, deterministic.

---

## The 5 evaluation heuristics

These are condition-agnostic detection patterns applied before scoring. They identify what's actually on the plate; the profile decides whether it's a problem.

1. **Healthy-sounding trap detection.** A name with *salad / bowl / wrap / protein / wellness / lite / fit* does NOT lower the fat/sugar estimate. Canonical traps: Caesar salad (bacon + parmesan + creamy dressing), Cobb salad (bacon + blue cheese + avocado + egg), wellness/poke bowls (tempura crisps + mayo sauces + imitation crab), granola bowls (high sugar + coconut + nut butter), açai bowls (concentrated fructose + nut butter + coconut), avocado toast variations.

2. **Hidden-fat sources.** Always check for: oil absorbed in stir-fry / fried rice, aroma oil at the bottom of brothless noodles (mazesoba — 2–3 tbsp), butter in "meunière" / pan sauces, mayo-based "special / spicy / aioli" sauces, cream in alfredo / carbonara / cacio e pepe / 4-formaggi / truffle, coconut milk in curries, deep-fried garnishes (tempura crisps, crispy garlic/onion), cheese melts, bone marrow, foie gras, olive oil drizzle at restaurant volumes.

3. **Broth discrimination.** Clear / consommé-style (pho, tom yum clear, shio/shoyu ramen, miso, chicken consommé) = light. Emulsified / creamy (tonkotsu, tom kha, bisque, chowder, paitan) = heavy. When unclear, ask or flag.

4. **Cut / protein discrimination.** Same animal, different cut = very different fat. Lean: tenderloin / sirloin / breast / white fish / shellfish. Fatty: ribeye / brisket / belly / chicken thigh+skin / wagyu / otoro. Special: fatty fish (salmon, sardines, mackerel) = beneficial omega-3 → positive bonus.

5. **Cooking method ladder.** From light to heavy: `raw / poach / steam / boil → grill (dry) → braise / light sauté → bake → pan-fry → deep-fry`. Profile decides how much each step weighs.

---

## Honesty rules (the contract)

Fourteen non-overridable rules. The ones you'll feel most:

- **Never soften a red score.** Red is red, reasoning shown on request only, color stays.
- **STRICT vs PRAGMATIC.** In STRICT mode (acute flare), no ranking of bad options — redirect. In PRAGMATIC mode, ranking allowed but must explicitly flag the top option as suboptimal if it's not 🟢.
- **Modifications must be physically realistic.** No "carbonara without cheese", no "alfredo without cream", no "risotto without rice". The kitchen can't do these without breaking the dish.
- **Tag every health claim** with an evidence level — 🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate. Untagged health claims are not allowed.
- **Anti-disorder safeguards (ED-safe mode).** If history or in-session signals suggest disordered eating — soften output, use ranges not point numbers, balance-focused framing, refer to a professional on red flags.
- **Wellness-label skepticism.** "Sugar-free / natural / protein / lite / clean / plant-based" never auto-raises a score. Read the actual composition.
- **Indulgence framework only in PRAGMATIC.** STRICT means redirect.
- **Output format compliance is mandatory.** Mode A and Mode B templates are contracts, not suggestions. Narrative prose where a table is specified = violation.
- **Language compliance is mandatory.** Use the `communication_language` from `profile.md`. Dish/menu names preserved verbatim from input. No mid-message-drift between English and Russian.

---

## Supported conditions

The skill is **condition-agnostic** — there are no hardcoded condition tables in `SKILL.md`. At first onboarding, you describe your condition in your own words; the skill matches to a row in its Source Map and fetches current guidelines.

| Condition | Preferred sources |
|---|---|
| Gallbladder / biliary / gallstones | EASL Gallstone Guidelines (2016); Mayo Clinic gallstones; Stol №5 |
| Gastritis / peptic ulcer / GERD | ACG; Mayo Clinic; Stol №1 |
| Diabetes (T1, T2, glycemic control) | ADA Standards of Care; Mayo Clinic |
| Hypertension | DASH / NHLBI; AHA |
| Dyslipidemia / high cholesterol / high LDL | ESC/EAS Dyslipidaemia; AHA |
| Kidney / CKD | KDIGO; NKF; Mayo Clinic |
| IBS | Monash FODMAP; ACG IBS; NICE |
| Gout | ACR; EULAR; Mayo Clinic |
| NAFLD / MASLD / fatty liver | AASLD; EASL NAFLD; Mayo Clinic |
| Weight loss | DGA; Mediterranean; ACSM |
| Muscle gain | ISSN Sports Nutrition; ACSM |
| Healthy eating / prevention | DGA; WHO; Mediterranean |
| Anything else | Mayo Clinic + EFSA + WHO + USDA |

If your condition isn't on the list, the skill derives 3–5 priority parameters from the relevant guideline (Mayo Clinic / EFSA / WHO baseline) and tells you upfront which it will weigh heaviest.

The skill explicitly surfaces source conflicts — e.g. for asymptomatic gallstones, EASL doesn't require strict fat restriction while Stol №5 does. You see both, and a recommendation suited to your specific situation (phase, structural risk factors).

---

## Installation

### Claude Code (recommended — persistent profile, WebFetch works)

```bash
git clone https://github.com/stukhin/food-analyzer-skill.git ~/.claude/skills/food-analyzer
```

Restart Claude Code (or just start a new session). The skill activates automatically when you send a food photo, mention a dish, or ask "can I eat this".

**First session:** the disclaimer + language choice + 5-group wizard runs. After confirmation, profile is saved to `~/.claude/skills/food-analyzer/profile.md`.

**Subsequent sessions:** profile auto-loads. The skill confirms ("Profile loaded: gallbladder, remission. Confirm or update.") and is ready in your chosen `communication_language`.

### Claude.ai (web or desktop app)

1. Create a new Project on claude.ai.
2. Add all skill files to Project Knowledge:
   - `SKILL.md`
   - All files in `cuisines/`
   - All files in `references/`
3. In Project Custom Instructions, add:
   > *Use the protocol in the attached SKILL.md exactly. Follow Honesty Rule 13 (output format compliance) and Honesty Rule 14 (language compliance). Ask the user explicitly which language to use (English or Russian) before starting the intake wizard.*
4. Open a new chat inside this Project. Send a food photo.

**Limitation on Claude.ai:** no persistent `profile.md` (no file system access), so onboarding runs each session. WebFetch may or may not work depending on your plan — if not, the skill falls back to its most recent training-data snapshot of the guideline and flags this in output.

---

## Privacy

- `profile.md` is **gitignored** — your medical data never leaves your machine via this repo.
- `backups/` is **gitignored** — historical profile snapshots stay local.
- `guidelines_cache.md` ships empty in the repo skeleton; it populates locally from your WebFetch calls.
- The skill explicitly **does not** read from Claude's broader auto-memory (`MEMORY.md` or project memory) to determine your condition. It only uses what you've explicitly confirmed at onboarding. This is documented as the **Memory boundary** rule in `SKILL.md`.

If you share this skill with someone (e.g. via a zip of the repo), they get a clean install — no traces of your profile.

---

## Roadmap

**v3.2.0 (current)** — explicit bilingual support. Language selection at onboarding, foreign-content preservation, full English documentation with Localization table, Honesty Rules 13–14 enforcing format and language compliance.

**v3.3 (planned)** — full Mode C mechanics:
- Variety enforcement (no identical consecutive days, cuisine rotation)
- Shopping list generation for week-long plans
- Multi-day running totals with budget overflow detection

**v3.4+ (planned)** — additional cuisines (Chinese, Korean, Indian, Vietnamese, Mexican, Filipino, Levantine). Each follows the `_template.md` structure.

**Future** — auto-tuning of variable-size modifiers from the calibration log (only with user confirmation per Honesty Rule). Phase profile switcher ("switch to acute") that swaps active threshold sets without re-onboarding.

---

## Contributing

The skill is built for individual customization, but improvements to the core methodology, the cuisine knowledge base, and the source registry are welcome.

**To add a cuisine:**
1. Copy `cuisines/_template.md` to `cuisines/<your_cuisine>.md`.
2. Fill in profile summary, 8–14 standard dishes with components / typical macros / `modifications_possible`, red flags, safe choices, sources.
3. Keep it condition-agnostic. The profile decides 🟢/🟡/🔴; the cuisine file records raw facts only.
4. Open a PR with at least 2 example dishes scored against a couple of common profiles in the PR description.

**To add a source:**
1. Edit `references/_sources.md`, add a block following the existing format (name / domain / coverage / URL / authority type).
2. Update the Source Map in `SKILL.md` to include the condition keyword(s) that route to it.

**To suggest a heuristic:**
Open an issue describing the trap or hidden pattern with 2–3 example dishes. If it generalizes across cuisines and isn't already covered by the 5 existing heuristics, we add it.

---

## License

MIT — use freely, modify freely, share freely.

See [LICENSE](LICENSE) if present, otherwise refer to standard MIT terms.

---

**Repo:** https://github.com/stukhin/food-analyzer-skill
**Issues / questions:** open an issue on GitHub.
**Author:** Alexander Styukhin ([@stukhin](https://github.com/stukhin)). Built collaboratively with Claude.
