# TODO 
Rule: **only what the PAP specifies.** Anything that needs a judgement call goes under *Open questions* and is decided together before it is coded.

**Design in one line (PAP 3, 5.1):** individual-level RCT, 4 arms (Control, T1 CV, T2 CV+Recs, T3 CV+Recs+In-demand), strata = municipality × above/below-median age × program (Fomentar/VAT).
Spec: `Y = b0 + b1·CV + b2·SkillRecs + b3·DemandRecs + strata FE + X(double-selection LASSO) + e`.

---

## 1. Merge — `code/cleaning/merge.ipynb` → `data/intermediate/`
- [x] 0. Setup, paths, helper asserts
- [x] 1. Base = `individ_randomization.dta` (1,034,347 persons; `id` = `id_SSE`; arms balanced at 258,586–258,587)
- [x] 2. Crosswalks: SkillLab `user_id` → `id_correo` → `id_SSE`; ambiguous e-mails (11) are *not* linked
- [x] 3. Ministry flags (files 1–4, `receiving_benefits_variable`) joined 1:1 on `id_SSE`
- [x] 3b. Ministry employment panel (person-month, 2024-01 to 2025-07) saved as long file
- [x] 4. SkillLab: event sheets → user level → person level (usage counts, in-app survey); event tables saved for alignment measures
- [x] 5. Opinaia phone survey (`Sheet1`, 1,021 respondents; names/phones dropped before saving)
- [x] 6. Assemble person file with `validate="1:1"` and row-count asserts; coverage diagnostics by arm
- [x] 7. Save: `merged_person.csv`, `emp_panel_long.csv`, `skilllab_*_events.csv`
- Not merged on purpose: `labormarket_lightcast-selected/` (country-month aggregates, no individual key; not used in the PAP regressions)

## 2. Clean — `code/cleaning/clean.ipynb` → `data/clean/`
- [x] 1. Treatment dummies `CV`, `SkillRecs`, `DemandRecs` (reference = Control)
- [x] 2. Strata ID (municipality × age-above-median × program)
- [x] 3. Heterogeneity variables: `female`, `edad`, `edad_above_med`
- [x] 4. Pool of candidate controls for double-selection LASSO (31 variables)
- [x] 5. Survey outcomes mapped to PAP families + control-SD standardization
- [x] 6. Admin outcomes: course enrollment, CV update, formal employment (post window)
- [x] 7. SkillLab take-up variables
- [x] 8. Checks (unique ids, no control sign-ups, balance) and save `analysis_full.csv`, `analysis_survey.csv`, `analysis_vars.json`

## 3. Analysis (not started — next step after the open questions are settled)
**Main specification (PAP 5.1)**
- [ ] Double-selection LASSO of controls `X` for each outcome (pool: `x_candidates` in `analysis_vars.json`; strata FE always included)
- [ ] Regress each outcome on `CV`, `SkillRecs`, `DemandRecs` + strata FE + selected `X`; robust SEs
- [ ] RQ1: Control vs T1 — `H0: b1 = 0`
- [ ] RQ2: Control vs T2 — `H0: b2 = 0`
- [ ] RQ3: T1 vs T2 — `H0: b1 = b2`
- [ ] RQ4: Control vs T3 — `H0: b3 = 0`
- [ ] RQ5: T2 vs T3 — `H0: b2 = b3`
- [ ] Outcome families (PAP 4.3): (1) skills knowledge & career aspirations, (2) job search, (3) reservation wage, (4) employment, (5) mental health
- [ ] Administrative outcomes (PAP 4.2): course enrollment/completion, formal employment (wage, sector), CV updates

**Heterogeneity (PAP 5.2)**
- [ ] Interact treatments with gender (`female`) and with age (`edad_above_med`)
- [ ] Home-to-course and home-to-job distances by arm; distance heterogeneity by gender (blocked, see Open questions)

**Multiple hypothesis testing (PAP 5.3)**
- [ ] FDR q-values within each outcome family

**SkillLab usage / alignment (PAP 4.1)**
- [ ] Alignment of career interests with own skills; of liked/viewed courses with skills; of interests with in-demand careers (T2 vs T3)
- [ ] In-app confidence survey (T2/T3 only), skill-laddering (Ministry courses + SkillLab skills)

**Reporting**
- [ ] Take-up table by arm; balance table; main-results tables per family; power check vs. PAP MDEs (0.075–0.225 SD)

---

## Open questions — gaps between the PAP and the data in hand (need a decision, nothing was changed silently)
1. **Mental health** (PAP outcome family 5): the Opinaia questionnaire contains no mental-health items. Is there another survey wave/file?
2. **Midline vs. endline**: PAP plans two phone surveys (~3 and ~9 months after access). `2025.08.06 Base.xlsx` is one wave with 1,021 respondents (100 Control / 320 / 301 / 300), vs. the 5,000 planned in the PAP. All respondents are in `treat_phase = 2`. Is more survey data coming?
3. **Estimation sample**: the randomization file assigns arms to 1,034,347 people, but the PAP says 500,000 invitations went out, and the e-mail crosswalk only contains ~62,500 invited people (`invited = 1`). ITT on all randomized, on invited only, or by `treat_phase`?
4. **SkillLab linkage / take-up**: only 1,279 of 3,363 sign-ups link to an `id_SSE` (the `user_correo` crosswalk covers 1,385 users). Observed take-up is ~0.16% of assigned people, far below the 5–15% in the PAP's power calculation. Missing crosswalk rows for the other sign-ups?
5. **Strata with missing municipality**: `muni_id` is missing for 158,683 people (15.3%, evenly across arms). Strata cannot be formed for them; they are currently left out of strata FE (not imputed). All 1,021 survey respondents have a municipality (13 municipalities).
6. **Absent ids in Ministry files 1–4** are coded 0 (the files only list `1`s). Please confirm with the Ministry.
7. **Employment panel covers only 6,026 ids** (~0.6% in each arm; no wage or sector fields, just employer name and location). Ids outside the panel are kept as missing, not as unemployed. Is the panel a sample or an extract of the formally employed? PAP also mentions **wage and sector** — not in the data.
8. **"Post" window for employment**: the PAP gives no cut-off. `POST_START_MONTH = 202505` (first SkillLab sign-up is 2025-04-30) is a placeholder in `clean.ipynb`; the panel ends 2025-07.
9. **Distances (PAP 5.2)**: no course location/modality or firm location file in `data/raw/` (and no municipality coordinates). Needed for the mobility analysis.
10. **Control pool for LASSO**: the PAP says "socio-demographic controls"; the exact pool is not listed. Current pool = variables in the registration data (see `clean.ipynb` section 5). Confirm.
11. **Data errors reported, not fixed**: one person has `edad = 2025` (probably a birth year). `edad_above_med` was built by the research team and is used as given.
12. **Outcome definitions from the survey**: the mapping of P0–P10 to the PAP families (clean.ipynb section 6) is my reading of the questionnaire; the PAP only lists the families. Standardization uses the control-group SD within respondents.
13. **Course completion** (PAP 4.2) is not in the Ministry files (only `enrolled_in_course`).
