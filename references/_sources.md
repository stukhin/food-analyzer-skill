# Authoritative Sources Registry

A registry of authoritative bodies the food-analyzer skill may cite. Used as a lookup when synthesizing thresholds at onboarding and when tagging health claims with evidence levels.

Format per entry: name · domain · what it covers · primary URL · type of authority.

---

## Clinical guideline bodies

### EASL — European Association for the Study of the Liver
- Domain: liver and biliary diseases
- Covers: gallstones (EASL CPG 2016), NAFLD/MASLD, cirrhosis, viral hepatitis
- URL: https://easl.eu/publications/clinical-practice-guidelines/
- Type: 🔵 Guideline

### AASLD — American Association for the Study of Liver Diseases
- Covers: NAFLD/MASLD (AASLD Guidance), hepatocellular carcinoma, autoimmune hepatitis
- URL: https://www.aasld.org/practice-guidelines
- Type: 🔵 Guideline

### ACG — American College of Gastroenterology
- Covers: gastritis, peptic ulcer, GERD, IBS, IBD, celiac, pancreatitis
- URL: https://gi.org/clinical-guidelines/
- Type: 🔵 Guideline

### ADA — American Diabetes Association
- Covers: type 1 and type 2 diabetes, prediabetes, gestational diabetes
- Annual flagship: Standards of Medical Care in Diabetes
- URL: https://diabetesjournals.org/care/issue
- Type: 🔵 Guideline

### AHA — American Heart Association
- Covers: dietary guidelines for cardiovascular disease prevention, sodium, saturated fat, cholesterol
- URL: https://professional.heart.org/en/guidelines-and-statements
- Type: 🔵 Guideline

### ESC/EAS — European Society of Cardiology / European Atherosclerosis Society
- Covers: dyslipidaemia management, lipid targets
- URL: https://www.escardio.org/Guidelines/Clinical-Practice-Guidelines
- Type: 🔵 Guideline

### NHLBI / NIH — National Heart, Lung, and Blood Institute
- Covers: DASH diet, hypertension management, weight management
- URL: https://www.nhlbi.nih.gov/health-topics/dash-eating-plan
- Type: 🔵 Guideline

### KDIGO — Kidney Disease Improving Global Outcomes
- Covers: CKD nutrition, dialysis nutrition
- URL: https://kdigo.org/guidelines/
- Type: 🔵 Guideline

### NKF — National Kidney Foundation
- Covers: practical CKD diet guidance
- URL: https://www.kidney.org/atoz/atozTopic_Diet
- Type: 🔵 Guideline

### ACR — American College of Rheumatology
- Covers: gout management (ACR Gout Guideline)
- URL: https://www.rheumatology.org/Practice-Quality/Clinical-Support/Clinical-Practice-Guidelines
- Type: 🔵 Guideline

### EULAR — European Alliance of Associations for Rheumatology
- Covers: gout, rheumatic disease nutrition
- URL: https://www.eular.org/recommendations
- Type: 🔵 Guideline

### NICE — UK National Institute for Health and Care Excellence
- Covers: broad clinical guidance including IBS, diabetes, NAFLD
- URL: https://www.nice.org.uk/guidance
- Type: 🔵 Guideline

### ISSN — International Society of Sports Nutrition
- Covers: protein requirements, muscle gain, performance nutrition
- URL: https://www.sportsnutritionsociety.org/position-stands.html
- Type: 🔵 Guideline (sports nutrition)

### ACSM — American College of Sports Medicine
- Covers: physical activity, weight management
- URL: https://www.acsm.org/education-resources/clinical-resources
- Type: 🔵 Guideline

---

## Specialty / topic-specific

### Monash University FODMAP Program
- Covers: low-FODMAP diet for IBS
- URL: https://www.monashfodmap.com
- Type: 🔵 Guideline (the authoritative source for FODMAP)

### Stol №1 / №5 / №9 (Pevzner diets)
- Origin: Russian/Soviet dietary tradition (Manuil Pevzner, 1920s–1950s)
- Coverage:
  - Stol №1 — gastritis, peptic ulcer
  - Stol №5 — gallbladder, liver, biliary
  - Stol №9 — diabetes
- **Marker:** Useful as a regional reference, often stricter than Western guidelines. Always mark explicitly as "Russian dietary tradition", not as universal medical consensus.
- Type: 🔵 Guideline (regional)

### DGA — Dietary Guidelines for Americans (USDA + HHS)
- Covers: general healthy eating, life stages
- URL: https://www.dietaryguidelines.gov
- Type: 🔵 Guideline

### Mediterranean Diet Foundation
- Covers: Mediterranean dietary pattern
- URL: https://dietamediterranea.com/en/
- Type: 🔵 Guideline (pattern)

### WHO — World Health Organization
- Covers: global nutrition guidance, daily intake reference
- URL: https://www.who.int/health-topics/nutrition
- Type: 🔵 Guideline (global)

### EFSA — European Food Safety Authority
- Covers: nutrient reference values for EU, food safety
- URL: https://www.efsa.europa.eu/en/topics/topic/dietary-reference-values
- Type: 🔵 Guideline (EU)

---

## Macro and composition references

### USDA FoodData Central
- Domain: nutrient composition of foods
- Use: per-100g macro lookups for ingredients and standardized recipes
- URL: https://fdc.nal.usda.gov
- Type: ⚪ Reference (not guideline, but authoritative composition data)

### Mayo Clinic patient resources
- Coverage: accessible explainers across most conditions
- URL: https://www.mayoclinic.org/diseases-conditions
- Type: ⚪ Clinical (institution-vetted patient education)

---

## How this registry is used

- The skill's `Source Map` in SKILL.md routes a stated condition to one or more entries here.
- At onboarding, the skill calls `WebFetch` against the URL(s) for the matching entries to pull current text.
- The fetched summary is cached in `guidelines_cache.md` with a timestamp.
- Cached entries refresh every ~90 days on next relevant use.
- Every threshold in profile.md cites the entry it derives from.

**Adding a new source:** Add a block following the format above. Include the URL, type (🔵 Guideline / 🟣 Evidence / ⚪ Clinical / ⚫ Estimate), and what it covers. If it's a regional or niche reference, mark that explicitly.
