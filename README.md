# 🥗 Food Analyzer Skill for Claude

A Claude skill that scores food photos, menus, and dish descriptions against **your personal medical condition or dietary goal** — using current published guidelines as the evidence base, not vibes.

> *"I tested it on a Hawaiian Loco Moco, expecting it to call it bad. It found four independent fat sources I hadn't noticed (mac salad is mayonnaise on macaroni, the gravy is built on rendered beef fat, the wonton crisps are deep-fried, the patty is pan-fried), scored it 1/10 🔴 with the actual per-meal target cited from EASL, and showed me a modified version that would land at 6/10 🟡. I switched my order."*

Works in **English** and **Russian** (auto-detects from your first message). Single-user skill (one install = one profile). Public, MIT-licensed.

[Install](#installation) · [Live examples](#-live-examples-text-based) · [Architecture](#-architecture) · [How scoring works](#how-scoring-works) · [Supported conditions](#supported-conditions)

---

## What's new in v3.1.0

The big one: **strict output-format compliance**. In v3.0 the skill applied the right heuristics but drifted into narrative prose in some Claude.ai sessions. v3.1 makes the table-based output structure a hard contract (Honesty Rule 13), with explicit WRONG-vs-RIGHT example and a pre-send checklist. Everything else from v3.0 (knowledge-base architecture, 6 cuisines, indulgence framework, evaluation heuristics, etc.) carries forward unchanged.

Full version history in [CHANGELOG.md](CHANGELOG.md).

---

## What it does (one paragraph)

You install the skill. At first use, it runs a five-group wizard that captures your basics, diagnosed conditions, current phase (acute/remission/preventive), goals, restrictions (medical/religious/ethical/personal), and personal calibration (foods that have given you symptoms, foods that feel safe). It looks up current guidelines for your condition(s) via WebFetch (EASL for gallstones, ADA for diabetes, ACG for gastritis, DASH for hypertension, ESC/EAS for dyslipidemia, KDIGO for kidney, ACR for gout, AASLD for fatty liver, Monash for IBS, DGA for general healthy eating — full source map below). It proposes a personalized threshold setup with non-overlapping value bands. You confirm or refine. From then on, every food photo or menu you send gets evaluated against your derived profile, producing a deterministic 1–10 score per parameter with stoplight color, a verdict, modification suggestions with explicit re-scoring delta, and a sensitivity disclosure.

The skill never claims medical authority. Every health-related claim is tagged with an evidence level — 🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate — and on red verdicts it appends a one-line disclaimer.

---

## 🎬 Live examples (text-based)

These are actual outputs the skill produces, copied as Claude would render them in your chat. No screenshots — text only, so you can read every detail.

### 1. First-time onboarding (compressed walkthrough)

> **User:** оцени мне это блюдо [photo attached]
>
> **Skill:** I'm a food-analysis tool, not medical advice. My scoring is based on published dietary guidelines — it doesn't replace your physician or dietitian. For serious conditions, consult a specialist.
>
> Quick onboarding before I score anything — five short groups, one at a time.
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
> - Medications / supplements that affect digestion (Ursofalk/Ursoliv, Creon, pancreatic enzymes, PPIs, statins)?
> - Current phase — *acute / postoperative / diagnostic* (→ STRICT mode), or *stable remission / general prevention* (→ PRAGMATIC mode, default)?
>
> **User:** gallstones (4 stones, 0–5mm, asymptomatic, found incidentally), slightly tortuous common bile duct, chronic gastritis but no symptoms. Stable. No meds.
>
> **Skill:** Two conditions — I need to know the priority before researching: which one drives my primary parameters right now? Gallstones with tortuous CBD sounds structurally riskier; gastritis is currently silent. Confirm or correct.
>
> **User:** да, желчный приоритет
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
> - Anything in your history that should change how I report numbers (history of eating disorder, obsessive macro tracking, restrictive escalation)?
>
> **User:** rich cream sauces gave dull right-side ache once; grilled fish + rice always feels good. No ED history.
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
> **Note on a source conflict:** EASL does NOT require strict fat restriction for asymptomatic gallstones (recommends moderate fat, omega-3 beneficial). Stol №5 is stricter (70–80g/day total fat, distributed dribble-style 4–5 meals). Given your tortuous CBD + actual stones (not just sludge), I'm taking a pragmatic middle — closer to Stol №5 on fat *type* and cooking method, looser on absolute grams since you're asymptomatic. Surfacing both so you know.
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
> **User:** Подтверждаю.
>
> **Skill:** Profile saved to `~/.claude/skills/food-analyzer/profile.md`. Backup at `backups/profile.md.backup-20260524-153012`. Кидай блюдо.

This whole conversation runs once. Subsequent sessions skip the wizard and just confirm the loaded profile in one line.

---

### 2. Mode A — Single dish, single source of complexity (Hawaiian Loco Moco)

The actual format the skill is required to produce for every single-dish evaluation:

```
**Блюдо:** Loco Moco (Hawaiian comfort plate)
**Уверенность:** макро M · готовка H · порция L · соус/состав L

Говяжья котлета (пан-фрай) + белый рис + грибная грэви + пашот-яйцо + mac salad
(макароны в майонезе) + вонтон-крисы (фритюр). Триггеры heuristic'ов: #1
(healthy-sounding trap не применим — mac salad звучит безобидно, но это
майонез на макаронах, не салат), #2 (hidden fat — грэви построена на жире
после обжарки котлеты + вонтоны деep-fried).

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

**Вердикт:** не брать — четыре независимых жировых источника в одном приёме,
2.5× превышение per-meal target по EASL [🔵 Guideline].

**Почему этот балл:** стекаются четыре независимых жировых источника
(котлета пан-фрай, mac salad на майонезе, грэви на жире, вонтон-крисы
фритюр), плюс готовка двойной удар. Любого одного хватило бы для жёлтого;
вместе — красный по первому Primary.

**Что можно изменить:**
- Plain steamed rice вместо garlic fried rice → −12g жира.
- Убрать вонтон-крисы → −10g жира.
- Grilled chicken breast вместо котлеты + грэви на сторону → −20g жира.

**Пересчёт с модификациями:** as served 1/10 🔴 → modified (grilled chicken
+ грэви на стороне + plain rice + без crisps) 6/10 🟡.

**Самое неопределённое:** объём грэви и заправки в mac salad (могут добавить
±10g жира).

⚠️ Не медицинская рекомендация. При острых симптомах — к врачу.

*Оцениваю в изоляции — скажи, что ел до этого, если хочешь дневной баланс.*

*Скажи, если что-то распознал не так.*
```

What this format guarantees: confidence breakdown by 4 aspects, macro ranges (not single numbers), a scored parameter table with bars, single-sentence verdict, reasoning inside `**Почему этот балл:**`, modifications with a concrete re-scored delta line, "most uncertain" call-out, mandatory disclaimer on red, dual footers.

---

### 3. Mode A — Healthy-sounding trap detection (Wellness Bowl)

User profile: T2 diabetes, stable, PRAGMATIC.

```
**Блюдо:** "Wellness Bowl" (label) / poke-style bowl with salmon
**Уверенность:** макро M · готовка M · порция M · соус/состав L

🚩 Healthy-sounding trap detected. Label "wellness" does not lower the
estimate: visible composition shows tempura crisps on top, "spicy mayo"
drizzle, imitation crab with mayonnaise, avocado, edamame, rice. Heuristic
#1 (trap detection) and #2 (hidden fat in two mayo-based sauces).

**Hidden fat sources:**
- Spicy mayo — ~2 tbsp = 12–18g fat (mostly saturated + processed)
- Imitation crab salad — typically with mayo, ~5–8g fat
- Tempura crisps — deep-fried, ~5–10g fat absorbed

**Макро (диапазон):**
- Калории: 620–820 kcal
- Белок:   24–32g
- Жир:     28–42g (насыщ. 6–9g)
- Углеводы: 65–80g (sugar 8–12g)

**Оценка:**

| Параметр      | Значение  | Балл | Бар         |
|---------------|-----------|------|-------------|
| Carbs         | 65–80g    | 4/10 | ████░░░░░░ 🟡 |
| Added sugar   | 8–12g     | 6/10 | ██████░░░░ 🟡 |
| Fiber         | 5–8g      | 8/10 | ████████░░ 🟢 |
| Sat fat       | 6–9g      | 7/10 | ███████░░░ 🟢 |
| Omega-3       | salmon    | +1   | бонус         |
| **Итого**     | —         | 5/10 | █████░░░░░ 🟡 |

**Вердикт:** caution — high total carb load plus hidden mayo-based fats
push it into yellow despite salmon and fiber being solid [🔵 ADA: ≤45g
carbs/meal target for T2].

**Почему этот балл:** salmon + edamame + avocado is a strong base, but
two mayo-based sauces and tempura crisps add 25g+ fat that doesn't appear
on the label. Carbs at 65–80g are also above ADA per-meal range. Omega-3
bonus and good fiber save it from red.

**Что можно изменить:**
- "Spicy mayo on the side" → −12–18g fat, score → 🟢
- "Hold the tempura crisps" → −5–10g fat
- "Imitation crab replaced with extra edamame" → −5–8g fat, +protein quality

**Пересчёт с модификациями:** as served 5/10 🟡 → modified (sauce on side
+ no tempura + edamame for imitation crab) 8/10 🟢.

**Самое неопределённое:** actual mayo volume. If drizzle = 1 tsp vs 2 tbsp,
the whole dish swings 🟡 → 🟢 without any modifications.

**Lesson:** "wellness / bowl / protein" in a name is not evidence. Check
sauces and toppings.

*Оцениваю в изоляции — скажи, что ел до этого, если хочешь дневной баланс.*

*Скажи, если что-то распознал не так.*
```

---

### 4. Mode A — Correction flow with re-scoring

After the Wellness Bowl was scored, the user says:

> **User:** actually it was half a bowl, the rest got boxed for tomorrow

The skill recalculates:

```
Понял — пересчитываю.

**Макро (пересчитано на половину порции):**
- Калории: 310–410 kcal
- Жир:     14–21g (насыщ. 3–5g)
- Углеводы: 32–40g

**Оценка:**

| Параметр    | Значение  | Балл | Бар         |
|-------------|-----------|------|-------------|
| Carbs       | 32–40g    | 8/10 | ████████░░ 🟢 |
| Sat fat     | 3–5g      | 9/10 | █████████░ 🟢 |
| Fiber       | 3–4g      | 7/10 | ███████░░░ 🟢 |
| **Итого**   | —         | 8/10 | ████████░░ 🟢 |

**Вердикт:** хороший выбор в новом объёме — половина порции укладывает
тебя в ADA per-meal targets по всем осям.

**Почему этот балл:** размер порции — единственное, что изменилось.
Композиция (соусы, темпура) осталась той же, но абсолютные граммы упали
вдвое, и параметры переехали в зелёное. Логировать как successful behavioral
modifier для этого формата.

Записал в калибровочный лог: *2026-05-26 — wellness bowl half portion —
ok, no symptoms expected based on composition*.

*Скажи, если что-то распознал не так.*
```

Note what didn't change: the skill did NOT lower the per-100g composition score (per Honesty Rule 4). It re-estimated the serving size, which legitimately moves absolute macros into target range.

---

### 5. Mode B — Multi-dish ranking (PRAGMATIC mode)

User sends a screenshot of five dishes from a Macro Bowls menu. Profile: gallbladder priority, PRAGMATIC.

```
Tu пять блюд из одного меню, ранжирую под твой профиль. Все на белом рисе
+ овощи на пару, поэтому разница в белке и соусе.

| # | Блюдо                | Цена | Жир (10-cell bar)   | Score    |
|---|----------------------|-----:|---------------------|:--------:|
| 1 | Chicken Oyakudon     | ₱280 | █░░░░░░░░░ 12–18g   | 8/10 🟢   |
| 2 | Eggs in Tomato       | ₱260 | ██░░░░░░░░ 15–22g   | 6/10 🟡   |
| 3 | Ham Omelette         | ₱290 | ████░░░░░░ 25–32g   | 5/10 🟡   |
| 4 | Chicken Salpicao     | ₱320 | █████░░░░░ 30–38g   | 4/10 🟡   |
| 5 | Chicken Ala King     | ₱350 | ████████░░ 40–55g   | 2/10 🔴   |

## Топ-выбор — почему
- **Chicken Oyakudon (8/10 🟢)** — grilled chicken breast + lightly cooked
  egg in soy-mirin sauce. Готовка гриль (+0), соус не на жире.
  Единственный флаг — sugar/salt в терияки по гастриту, но он немой.
  Самый чистый по желчному из пяти.

## Худшее — почему обойти
- **Chicken Ala King (2/10 🔴)** — описание выдаёт сразу: *"rich, creamy
  sauce made with cream, butter"*. Сливки + сливочное масло = прямой
  fat-shock penalty (−2), saturated fat в красной зоне (>12g на порцию).
  Классическая ловушка "здоровых macro bowls": grilled chicken и steamed
  vegetables маскируют сливочную базу. Жир 40–55g, красный по первому Primary.
- **Chicken Salpicao (4/10 🟡)** — salpicao традиционно делается на
  чесночно-масляной базе, плюс жареное яйцо. Грудка постная, но соус и
  яйцо вместе тянут жир к 30–38g.

## Что заказать
**Chicken Oyakudon, без модификаций.** Цена нормальная, по профилю это
чистый зелёный. Если хочется яиц — Eggs in Tomato, попроси меньше масла и
учти томат-кислоту (по гастриту терпимо в ремиссии). Chicken Ala King
пропусти, "grilled chicken + steamed veg" в описании не отменяет
сливочный соус.

*Confidence: средний по всем. Граммовку соусов не вижу, но составы и
типичные рецепты этих блюд однозначны.*

*Want a detailed breakdown for any of these?*
```

---

### 6. Mode B — STRICT mode (acute flare)

Same five dishes, but the user is now in an acute gallbladder flare. STRICT mode kicks in.

```
**STRICT режим — не ранжирую варианты, потому что все на жирной базе
несовместимы с острой фазой.**

Все пять блюд содержат либо grilled meat + соусы (Oyakudon, Salpicao,
Ala King, Ham Omelette), либо жареное яйцо (Eggs in Tomato). В acute
фазе при гастрите + желчном — это все 🟡/🔴 территория.

**Don't eat here right now.** A suitable place during your flare would offer:
- Clear-broth soups (miso, dashi, chicken consommé, lean vegetable soup)
- Steamed white fish (cod, sole, snapper, hamachi sashimi if you can do raw)
- Plain steamed rice + boiled chicken breast (white meat only)
- Soft-boiled or poached eggs (one yolk maximum per meal)
- Steamed/boiled vegetables without oil
- Avoid: anything fried, any cream/butter sauces, any tomato/citrus/vinegar,
  any spicy seasoning, coffee, alcohol.

If you can't find a suitable venue, eat at home today. Once you're back
to remission, this menu has workable options — I'll re-rank then.

*Скажи, если что-то распознал не так, или если профиль фазы устарел и
ты уже в ремиссии.*
```

In STRICT, the indulgence framework is disabled (Honesty Rule 11). No "least bad" ranking.

---

### 7. Indulgence framework — treats in PRAGMATIC

User profile: dyslipidemia, stable, PRAGMATIC. They ask:

> **User:** Хочу мороженое раз в неделю. Можно?

```
**Можно ли это?** — да, но как риск-менеджмент, не "просто так". Мороженое
(full-fat ice cream, не sorbet) для твоего профиля проблематично по
концентрированному saturated fat (heavy cream + egg yolks).

**Что конкретно плохо здесь:**
- 1 шарик premium ice cream ≈ 8–14g жира, из них 5–9g saturated.
  ESC/EAS per-meal target <7g saturated [🔵 ESC/EAS].
- Сахар 12–22g/шарик — sweet-drink-like spike.

**Модуляторы риска (стэкуются):**
- **Один шарик, не два-три** — самый сильный lever, переводит 🔴 в 🟡.
- **После лёгкой еды**, не после стейка с pasta carbonara.
- **Не на голодный** — нужна еда в системе.
- **Раньше в дне** — после обеда лучше, чем после ужина.
- **15-минутная прогулка после** — поддерживает digestion и glucose.

**Красные комбинации не делать:**
- Большая порция (2–3 шарика) + после жирного ужина + поздно вечером —
  три "плохо" подряд.
- Мороженое + сладкий cocktail — двойной sugar load.

**Лёгкая альтернатива (та же ниша):**
- **Sorbet** (фруктовый лёд) — 0g сатурированного жира, сахар похожий,
  попадает в 🟡.
- **Dark chocolate gelato** небольшой шарик — saturated fat ~3g вместо 8g.
- **Frozen yogurt** — но **wellness-label trap, проверь состав**: "low fat"
  версия может быть ок, "creamy" версия = всё то же.

**Частота:** раз в 1–2 недели в перечисленных модуляторах — клинически
приемлемо для dyslipidemia в ремиссии [⚪ Clinical]. Чаще — кумулятивный
эффект на LDL.

⚠️ Не медицинская рекомендация.

*Скажи, если что-то распознал не так.*
```

---

### 8. Daily Tracker mode

Activated when the user mentions tracking the day. Profile: gallbladder.

> **User:** веду дневник, на завтрак был omelet с двумя яйцами + ломтик хлеба + чёрный кофе

```
**Daily Tracker on.** Limits I'm using for your profile:
- Калории: 1800 kcal
- Жир: 75g, дробно (~15-20g на приём, дневная цель Стол №5)
- Насыщ.: 20g
- Натрий: 2000mg

**Завтрак залогирован:**
- 2-egg omelet — ~14g fat (10g sat), 16g protein, 1g carb
- Toast — 80 kcal, 1g fat, 15g carb
- Чёрный кофе — 0 macro
- Итого завтрака: ~280 kcal, 15g fat, 17g protein, 16g carb

**Дневной баланс после завтрака:**
```
  Калории:  280 / 1800   (16%)   осталось 1520
  Жир:      15g / 75g    (20%)   осталось 60g
  Насыщ.:   10g / 20g    (50%)   ⚠️ осталось 10g
  Натрий:   180 / 2000   (9%)    осталось 1820
```

**Внимание на saturated fat:** уже 50% дневного лимита съел на завтраке
(yolks + butter в омлете). Обед и ужин держи на постном белке (fish, lean
chicken breast, legumes) — иначе вечером ляжешь над лимитом.

*Кидай следующее блюдо — посчитаю в контексте дневного баланса.*
```

After the next meal, the skill applies both per-meal scoring AND the budget layer:

```
**Блюдо:** Salmon Teriyaki Bowl
[... standard Mode A output with table ...]

**Per-meal verdict:** 7/10 🟢

**В контексте дня:**
- +14g жира → дневное 29g/75g (39%) ✓
- +3g насыщ. → дневное 13g/20g (65%) ✓ (вписывается)
- Omega-3 бонус от лосося ✓ — день балансируется

Verdict holds at 🟢 in today's context.

**Дневной баланс после обеда:**
[... updated budget table ...]
```

If saturated fat had pushed over the daily limit, the budget layer would shift verdict color independently of per-meal — this is the only mechanism in v3.1 allowed to cross color outside the per-meal lookup (Layer Separation rule).

---

### 9. Calibration log

After a Mode A evaluation, the user volunteers:

> **User:** после тех ребрышек вчера у меня болело в правом боку часа три, дискомфортно

The skill responds briefly:

```
Записал.

→ `~/.claude/skills/food-analyzer/profile.md > Personal calibration log`:
   - 2026-05-26 — fried pork ribs — right-side dull ache, ~3 hours

Не меняю пороги от одного наблюдения (шум). Если такая реакция повторится
2 раза подряд на похожих жирных блюдах, я подсвечу паттерн и предложу
ужесточить fat-shock модификатор.
```

After three consistent entries on similar triggers, the skill surfaces the pattern:

```
За последний месяц три раза dishes с >35g жира за один присест давали
тебе right-side ache 2–4 часа. Это твоя личная сигнатура fat-shock
триггера. Предлагаю:
- ужесточить per-meal fat cap с 35g → 30g (Primary band shifts)
- усилить fat-shock penalty с −2 на −3

Хочешь применить? Сделаю backup профиля, ребилд и покажу diff. Без
твоего "да" не трогаю.
```

The user controls all profile changes. The skill suggests, the user confirms.

---

### 10. Mode C — Meal planning (v3.0 minimal skeleton, full mechanics in v3.1+)

> **User:** что съесть сегодня

```
**Plan for today** (gallbladder priority + silent gastritis, PRAGMATIC,
~70-80g daily fat split across 4–5 meals).

### Завтрак
- Овсянка на воде + 1 банан + 1 tbsp миндальной пасты + черный кофе
- ~350 kcal, 10g fat (mostly mono-unsat), 50g carb, 9g protein
- Score: 8/10 🟢

### Обед
- Grilled salmon (150g) + brown rice (100g cooked) + steamed broccoli
- ~520 kcal, 18g fat (omega-3 dominant), 35g carb, 35g protein
- Score: 9/10 🟢, omega-3 bonus stacking

### Перекус
- Apple + 6 raw almonds
- ~140 kcal, 6g fat, 22g carb, 3g protein

### Ужин
- Chicken breast (120g) + quinoa (80g cooked) + roasted zucchini + tomato
- ~420 kcal, 8g fat, 38g carb, 32g protein
- Score: 9/10 🟢

**Дневной итог (расчётный):**
- Calories: ~1430 (maintenance ok)
- Жир: ~42g (well under 75g cap, distributed across 4 sittings — Stol №5
  dribble principle respected)
- Sat fat: ~7g (well under 20g cap)
- Omega-3 portion present at lunch ✓

**Что варьируется:** salmon на обед — основной омега-3 hit для дня. Если
повторил из вчера, поменяй на cod/sea bass + добавь грецкий орех в полдник
для омега-3. Quinoa на ужин — вариант, можно brown rice; cuisine rotation
gentle, не три однотипных дня подряд.

*Full Mode C mechanics (variety enforcement, shopping list, multi-day
running totals) — в v3.1+. Это v3.0 минимальный skeleton.*
```

---

## 🏛 Architecture

The skill is no longer a single file. It's a structured knowledge base under `~/.claude/skills/food-analyzer/`:

```
food-analyzer/
├── SKILL.md                    ← methodology (1300+ lines)
│                                  • workflows, output templates, 13 honesty rules,
│                                    13 anti-patterns, 5 heuristics, indulgence
│                                    framework, drinks hierarchy, etc.
│                                  • ZERO personal data, ZERO condition-specific
│                                    numbers
│
├── profile.md                  ← YOUR personal data + derived thresholds
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

**Key separation:** SKILL.md never holds your data; profile.md never holds methodology. Updating either one doesn't touch the other. Cuisines/references can be extended without touching either.

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

3. **Broth discrimination.** Clear/consommé-style (pho, tom yum clear, shio/shoyu ramen, miso, chicken consommé) = light. Emulsified/creamy (tonkotsu, tom kha, bisque, chowder, paitan) = heavy. When unclear, ask or flag.

4. **Cut / protein discrimination.** Same animal, different cut = very different fat. Lean: tenderloin / sirloin / breast / white fish / shellfish. Fatty: ribeye / brisket / belly / chicken thigh+skin / wagyu / otoro. Special: fatty fish (salmon, sardines, mackerel) = beneficial omega-3 → positive bonus.

5. **Cooking method ladder.** From light to heavy: `raw / poach / steam / boil → grill (dry) → braise / light sauté → bake → pan-fry → deep-fry`. Profile decides how much each step weighs.

---

## Honesty rules (the contract)

Thirteen non-overridable rules. The ones you'll feel most:

- **Never soften a red score.** Red is red, reasoning shown on request only, color stays.
- **STRICT vs PRAGMATIC.** In STRICT mode (acute flare), no ranking of bad options — redirect. In PRAGMATIC mode, ranking allowed but must explicitly flag the top option as suboptimal if it's not 🟢.
- **Modifications must be physically realistic.** No "carbonara without cheese", no "alfredo without cream", no "risotto without rice". The kitchen can't do these without breaking the dish.
- **Tag every health claim** with an evidence level — 🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate. Untagged health claims are not allowed.
- **Anti-disorder safeguards (ED-safe mode).** If history or in-session signals suggest disordered eating — soften output, use ranges not point numbers, balance-focused framing, refer to a professional on red flags.
- **Wellness-label skepticism.** "Sugar-free / natural / protein / lite / clean / plant-based" never auto-raises a score. Read the actual composition.
- **Indulgence framework only in PRAGMATIC.** STRICT means redirect.
- **Output format compliance is mandatory.** Mode A and Mode B templates are contracts, not suggestions. Narrative prose where a table is specified = violation.

---

## Supported conditions

The skill is **condition-agnostic** — there are no hardcoded condition tables in SKILL.md. At first onboarding, you describe your condition in your own words; the skill matches to a row in its Source Map and fetches current guidelines.

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

**First session:** the 5-group wizard runs. After confirmation, profile is saved to `~/.claude/skills/food-analyzer/profile.md`.

**Subsequent sessions:** profile auto-loads. The skill confirms ("Profile loaded: gallbladder, remission. Confirm or update.") and is ready.

### Claude.ai (web or desktop app)

1. Create a new Project on claude.ai.
2. Add all skill files to Project Knowledge:
   - `SKILL.md`
   - All files in `cuisines/`
   - All files in `references/`
3. In Project Custom Instructions, add:
   > *Use the protocol in the attached SKILL.md exactly. Follow the strict output-format compliance rules (Honesty Rule 13) — every Mode A response must use the full template structure with the parameter table and bars. Never narrate analysis as prose where a template is specified.*
4. Open a new chat inside this Project. Send a food photo.

**Limitation on Claude.ai:** no persistent `profile.md` (no file system access), so onboarding runs each session. WebFetch may or may not work depending on your plan — if not, the skill falls back to its most recent training-data snapshot of the guideline and flags this in output.

---

## Privacy

- `profile.md` is **gitignored** — your medical data never leaves your machine via this repo.
- `backups/` is **gitignored** — historical profile snapshots stay local.
- `guidelines_cache.md` ships empty in the repo skeleton; it populates locally from your WebFetch calls.
- The skill explicitly **does not** read from Claude's broader auto-memory (`MEMORY.md` or project memory) to determine your condition. It only uses what you've explicitly confirmed at onboarding. This is documented as the **Memory boundary** rule in SKILL.md.

If you share this skill with someone (e.g. via a zip of the repo), they get a clean install — no traces of your profile.

---

## Roadmap

**v3.1.0 (current)** — strict output-format compliance. Honesty Rule 13 enforces the template. WRONG vs RIGHT example pair added. Anti-pattern #13 documents narrative drift.

**v3.2 (planned)** — full Mode C mechanics:
- Variety enforcement (no identical consecutive days, cuisine rotation)
- Shopping list generation for week-long plans
- Multi-day running totals with budget overflow detection

**v3.3+ (planned)** — additional cuisines (Chinese, Korean, Indian, Vietnamese, Mexican, Filipino, Levantine). Each follows the `_template.md` structure.

**Future** — auto-tuning of variable-size modifiers from the calibration log (only with user confirmation per Honesty Rule). Phase profile switcher ("переключи на acute") that swaps active threshold sets without re-onboarding.

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
