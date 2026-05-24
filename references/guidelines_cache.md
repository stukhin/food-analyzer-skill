# Guidelines Cache

Per-condition and per-goal cached summaries from authoritative sources. Populated dynamically at user onboarding via `WebFetch`. **This file ships empty** — the skill populates it the first time a user names a condition.

---

## Cache format

For each cached entry:

```
## [Condition / Goal name]

### Source: [Source name from _sources.md]
- Fetched: YYYY-MM-DD
- URL: [exact URL fetched]
- Key thresholds:
  - [parameter 1]: [value, units]
  - [parameter 2]: [value, units]
  - ...
- Key recommendations: [2–4 bullets, paraphrased]
- Notable changes from prior version: [if known]

### Source: [Second source for the same condition]
- ...
```

If two sources for the same condition differ on a parameter, record both and let the skill surface the conflict to the user (per **Conflicting Sources** rule in SKILL.md).

---

## Refresh logic

- On every condition lookup, check the cached entry's `Fetched` date.
- If **≤90 days old** → use cached values, optionally mention the cache date.
- If **>90 days old** → `WebFetch` the URL again, update the entry, save new fetch date.
- If a guideline materially changed (different thresholds, new restrictions, removed recommendations), tell the user once: *"Guidelines updated since your last profile build — relevant changes: [...]. Want me to refresh your profile?"*

---

## Cache entries

*(initially empty — populated at first user onboarding)*

<!-- EXAMPLE PLACEHOLDER (not active, kept as format reference):

## NAFLD / MASLD

### Source: AASLD Practice Guidance on the Clinical Assessment and Management of NAFLD
- Fetched: 2026-05-24
- URL: https://www.aasld.org/practice-guidelines/clinical-assessment-and-management-nonalcoholic-fatty-liver-disease
- Key thresholds:
  - Weight loss target: 5–10% of body weight
  - Saturated fat: <10% of calories
  - Added sugar: minimize (especially fructose)
  - Coffee: 2–3 cups/day associated with reduced fibrosis [🟣 Evidence]
- Key recommendations:
  - Mediterranean diet pattern preferred
  - Eliminate sweetened beverages
  - Limit ultra-processed food
  - Moderate alcohol (or abstain if advanced fibrosis)
- Notable changes from prior version: none recorded yet

### Source: EASL NAFLD Clinical Practice Guidelines
- Fetched: 2026-05-24
- URL: https://easl.eu/wp-content/uploads/2018/10/EASL-EASD-EASO-CPG-NAFLD.pdf
- Key thresholds:
  - Saturated fat: <10% of calories
  - Sugar: limit, especially fructose
  - Fiber: ≥25–30g/day
- Conflict with AASLD: none material

-->

---

## Notes

- The cache is **per-user** (lives in this user's skill instance). Different users may have different cached entries depending on conditions asked about.
- The cache is **gitignored** so it stays local — fetched content from third-party guidelines should not be redistributed via git.
- If the user's network access is unavailable and the cache is empty for a condition → the skill falls back to its training-data snapshot and explicitly flags this in the onboarding output (`"Web tools unavailable, cache empty — using my most recent training-data snapshot of [source]. Verify when online."`).
