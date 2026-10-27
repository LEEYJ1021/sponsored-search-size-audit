# What Can a Conditional Advertiser-Size Statistic Show? Auditing Sponsored Search under Incomplete Observability in South Korea

**Replication code and documentation for the Structural Signal Irrelevance (SSI) audit**, the companion repository to the manuscript of the same title.

> **Status and how to read this repository.** This is a research repository documenting the analysis pipeline behind the manuscript. It is not a publication, and nothing here should be cited as a peer-reviewed result. Where wording differs, **the manuscript is the authoritative statement of results**. The repository covers one platform, one observation window and one linear conditional statistic. It makes **no fairness, discrimination or causal claim**: every coefficient is a conditional association, and the manuscript's central point is what such a statistic can and cannot support when the serving structure is only partly observable.

---

## Table of contents

1. [At a glance](#1-at-a-glance)
2. [Question and statistic](#2-question-and-statistic)
3. [Data and sample flow](#3-data-and-sample-flow)
4. [Design and evidentiary tiers](#4-design-and-evidentiary-tiers)
5. [Results](#5-results)
6. [Multiplicity audit](#6-multiplicity-audit)
7. [Design proposals for audit and disclosure](#7-design-proposals-for-audit-and-disclosure)
8. [What is and is not robust](#8-what-is-and-is-not-robust)
9. [Limitations](#9-limitations)
10. [Repository structure](#10-repository-structure)
11. [Manuscript-to-repository map](#11-manuscript-to-repository-map)
12. [How to reproduce](#12-how-to-reproduce)
13. [Data availability, disclosures and citation](#13-data-availability-disclosures-and-citation)

---

## 1. At a glance

| | |
|---|---|
| **Setting** | Commerce search-advertising platform in South Korea; data from a third-party advertising agency that manages sponsored-search accounts |
| **Window** | 23 August 2025 to 21 July 2026 |
| **Unit** | Advertiser-day (days with at least one click, so cost per click is defined) |
| **Panel** | 15,361 advertiser-days, 228 advertisers (highly unbalanced: 122 advertisers contribute a single day; maximum 193) |
| **Outcome** | log CPC (daily billed cost divided by daily clicks) |
| **Conditioning variable** | Standardised log(1 + daily cost) |
| **Size measure** | Standardised log(1 + all-time number of ad groups); constant within advertiser |
| **Statistic** | The SSI statistic c′: the coefficient on size in a linear regression of log CPC on spend and size, clustered by advertiser |

**Headline findings (all conditional associations, per standard deviation of size):**

| Quantity | Estimate | Reading |
|---|---|---|
| Unconditional size–CPC association | 0.00 (p = .998) | Size and CPC are unassociated |
| Spend-conditioned c′ (primary, advertiser-day weights) | −0.25 (SE 0.135; 95% CI −0.517 to 0.012; p = .062; wild cluster bootstrap p = .072–.074) | Negative but inconclusive |
| Same coefficient in the clicks model | +0.25 | Exact mirror: c′ is a cost-efficiency statistic, not a price schedule |
| Heterogeneity across campaign types | asymptotic Wald p = .023; wild cluster bootstrap p ≈ .25; BH-adjusted p = .081 | Does not survive resampling or multiplicity adjustment |
| Keyword-record linkage | 96.8% / 94.8% / 92.9% (website / power content / brand-new product) vs 0.0% / 0.7% (local business / shopping) | Serving-structure information is available for some types and nearly absent for others |

**One-line summary.** The spend-conditioned size coefficient is negative and inconclusive, mirrors a size–clicks association by construction, changes significance with weighting, influential accounts and the size measure, and cannot be read as evidence of price discrimination or of fairness. The two segments with positive point estimates link to almost no keyword-auction records and are too small to audit, so whether any segment is disadvantaged remains open.

---

## 2. Question and statistic

**Audit problem.** An outside auditor observes outcomes (cost, clicks) and a few advertiser attributes, not the rules that map one to the other. The simplest observable contrast is whether advertisers with comparable current spend obtain comparable outcomes regardless of size.

**Three levels kept apart throughout.**

1. **Observable statistic.** The coefficient on size in the regression of log CPC on spend and size. A non-significant coefficient does not establish conditional independence, because nonlinear relations are not detected and power may be low.
2. **Auditability.** Whether the statistic can be computed and read reliably for a segment depends on measurement, applicability of its premise, and sample size and stability.
3. **Fairness or discrimination judgement.** This requires causal, institutional and intent evidence that no observational statistic supplies. The repository makes no such claim in either direction.

**What the statistic measures.** Because CPC is billed cost divided by clicks, log CPC = log cost − log clicks holds by construction. The size coefficient in the CPC regression therefore equals, up to the log(1 + cost) approximation, the negative of the size coefficient in a regression of log clicks on the same regressors; with the exact log of cost the two sum to zero to numerical precision (|sum| < 10⁻¹³). The CPC–spend slope is not interpreted for the same reason.

**Three practical conditions** (developed from one case after the heterogeneity and sensitivity results were seen; not a priori principles):

| Condition | Meaning |
|---|---|
| **M** (measurability) | Behavioural signal and outcome measured completely, independently of size, and without an arithmetic link that changes the coefficient's meaning. Conversion and ROAS outcomes are excluded because tracking backfill lag plausibly correlates with size. |
| **S** (serving-structure applicability) | The statistic presupposes an auction- or bidding-based pricing arrangement for the segment. |
| **R** (reliability) | Stability to weighting, inference method, sample composition and size measure, and enough observations to discriminate between substantively different states of the world. |

---

## 3. Data and sample flow

Four linked tables are used: an ad-performance log, a campaign table, an ad-group table and a keyword table. Coverage declines across tables, most visibly for local-business and shopping campaigns. The data cannot separate a platform-side serving arrangement from incomplete export of records, which shapes the interpretation throughout. Registered-bid analyses could not be reproduced on the final panel and are not reported.

| Stage | n | Note |
|---|---|---|
| Ad-performance log | 321 advertisers | 19,373,916 records |
| Without ad-group metadata | 58 advertisers | No size measure; outside the analysis |
| Ad-group metadata table | 263 advertisers | 9,823 ad-group rows |
| Ad-group analysis frame, identified advertisers | 261 advertisers | 5,526 ad-group rows; 4,407 carry an advertiser identifier, 1,119 cannot be attributed |
| In metadata but not in frame | 2 advertisers | At most 19 advertiser-days (about 0.1% of the panel) |
| Positive cumulative cost (core sample) | 228 advertisers | 33 zero-cost advertisers excluded |
| Advertiser-day panel | 15,361 observations | Median 1, maximum 193 per advertiser; top 20 contributors account for 24.8% |

**Segments** (plurality-spend campaign type): website = 184 advertisers (11,894 advertiser-days); local business = 27 (1,306); shopping = 17 (2,161). No core advertiser has power content or brand/new product as its plurality category. "Local-business exposure" means any non-zero local-business spend (72 of 228 advertisers).

**Weighting.** Primary estimates weight each advertiser-day equally (a pooled regression on the daily panel). Equal-advertiser weights (1/nᵢ per advertiser-day) define a different estimand and are reported as a sensitivity analysis. Neither is the better measure of equal treatment.

---

## 4. Design and evidentiary tiers

| Tier | Contents | Status |
|---|---|---|
| **Tier 1 (primary)** | Pooled model, segment stratification, joint heterogeneity test, 25-test multiplicity audit | Fixed in the original pipeline before the sensitivity analyses |
| **Tier 2 (post hoc)** | Unconditional comparator and clicks check, wild cluster bootstrap, equal-weight estimation, influence and random-deletion comparisons, local-business-exposure exclusion, alternative size measures, price-margin equivalence | Prompted by results; every Tier 2 analysis that was run is reported, whatever its outcome |
| **Tier 3 (exploratory)** | Descriptive linkage patterns, spend-share interactions | Not interpreted as findings |

No downstream tier upgrades an upstream one.

**Inference rule (stated before results).** Inference is clustered by advertiser with a small-sample correction. The reference p-value for the pooled coefficient is the wild cluster bootstrap-t; the reference for heterogeneity is the bootstrap Wald test together with Benjamini–Hochberg adjustment. Asymptotic cluster-robust p-values are reported for comparison but are not decisive. Wild cluster bootstraps use Rademacher and Webb six-point weights, null-imposed residuals, 2,000 draws and a fixed seed (2026). Segment intervals additionally use t(G−1) critical values. Minimum detectable effects (MDE) at 80% power and 5% significance are 2.81 times the standard error. Equivalence is assessed by two one-sided tests (TOST) with margins expressed as percentage CPC differences.

**Replication note.** The spend-conditioning comparison was recomputed with an independent re-implementation on the final panel (standardising size across advertisers). It reproduces the sample and the primary result up to the third decimal (c′ = −0.255, SE 0.136, p = .062, against −0.253, SE 0.135, p = .062 in the original pipeline); the equal-weight coefficient is −0.359 versus −0.356. The differences reflect the standardisation convention and bootstrap draws, and no conclusion depends on them.

---

## 5. Results

### 5.1 Pooled association, with and without spend

| Model | Weights | Coefficient | 95% CI | p | Wild bootstrap p (Webb / Rademacher) |
|---|---|---|---|---|---|
| log CPC ~ size (unconditional) | Advertiser-day | 0.000 | [−0.271, 0.272] | .998 | .997 / .997 |
| log CPC ~ spend + size (SSI statistic) | Advertiser-day | −0.255 | [−0.522, 0.013] | .062 | .073 / .072 |
| log clicks ~ spend + size (check) | Advertiser-day | +0.255 | [−0.013, 0.522] | .062 | .073 / .073 |
| log CPC ~ size (unconditional) | Equal advertiser | +0.078 | [−0.098, 0.255] | .385 | .401 / .401 |
| log CPC ~ spend + size (SSI statistic) | Equal advertiser | −0.359 | [−0.524, −0.193] | < .001 | < .001 / < .001 |

*(Re-implementation on the final panel; the original pipeline reports −0.253 for the primary row.)* At equal daily spend, −0.253 corresponds to about 22% lower CPC, equivalently about 29% more clicks. Pooled MDE is about 0.38 log points (about 46% CPC difference per SD). TOST for margins of ±10% / ±20% / ±30% gives p = .877 / .698 / .471, so **equivalence is not established**. The size–spend slope implied by the reported coefficients is about 0.20 (derived, not separately estimated): larger advertisers spend more per day.

### 5.2 Stability of the pooled estimate

| Specification | Advertisers | c′ | p |
|---|---|---|---|
| Primary: ad-group count, advertiser-day weights | 228 | −0.253 | .062 (bootstrap .072–.074) |
| Equal advertiser weights (different estimand) | 228 | −0.356 | < .001 |
| Control for number of campaign types | 228 | −0.235 | .076 |
| Excluding 72 local-business-exposed advertisers | 156 | −0.499 | not reported |
| Excluding the 15 advertisers flagged in the original influence diagnostic | 213 | −0.398 | < .001 |
| Excluding top 5 / 10 / 15 by DFBETA (outcome-informed) | 223 / 218 / 213 | −0.407 / −0.460 / −0.513 | < .001 |
| Campaign count as size | 228 | −0.326 | .001 |
| Keyword count as size | 221 | −0.241 | .067 |

Exclusions chosen by their effect on the estimate are post hoc and outcome-informed. The informative comparison is random deletion: deleting 5, 10 or 15 advertisers at random leaves the mean estimate at about −0.252 but yields p < .05 in 15.7%, 22.5% and 25.6% of 5,000 draws. The sign is stable; significance is sensitive to sample composition. Size-measure coefficients are not comparable in magnitude across measures.

### 5.3 Heterogeneity across campaign types

| Segment | Advertisers | Advertiser-days | c′ (primary) | 95% CI (z) | p | MDE80 | Equal-weight c′ |
|---|---|---|---|---|---|---|---|
| Website | 184 | 11,894 | −0.279 | [−0.560, 0.003] | .052 | 0.404 | −0.353 |
| Local business | 27 | 1,306 | +0.312 | [−0.177, 0.801] | .211 | 0.726 | +0.210 |
| Shopping | 17 | 2,161 | +0.245 | [−0.089, 0.579] | .151 | 0.509 | −0.194 |
| Pooled | 228 | 15,361 | −0.253 | [−0.517, 0.012] | .062 | ≈ 0.38 | −0.356 |

The joint Wald test of the size × type interaction gives p = .023 (type-specific spend slopes). A wild cluster bootstrap of the same statistic gives p = .219–.257; under equal weights the Wald test gives p = .064 and bootstrap p ≈ .24–.25. In the 25-test audit the heterogeneity test (H2) has BH-, Holm- and Bonferroni-adjusted p-values of .081, .430 and .566. **The segment pattern is a descriptive lead, not a finding.** The shopping sign depends on weighting; the website estimate is negative under both weightings. MDE80 corresponds to about 107% (local business), 66% (shopping) and 50% (website) CPC differences per SD, so the two small segments cannot be audited with these data.

### 5.4 Keyword-record linkage (serving-structure information)

| Campaign type | Ad groups linked to keyword-auction records | Core advertisers with plurality in type | Core advertisers with any spend in type |
|---|---|---|---|
| Website | 96.8% | 184 | 202 |
| Power content | 94.8% | 0 | 13 |
| Brand/new product | 92.9% | 0 | 0 |
| Local business | 0.0% | 27 | 72 |
| Shopping | 0.7% | 17 | 24 |

Linkage establishes a difference in the **availability** of linked auction records. It does not establish how each type is served: a missing match may reflect a serving arrangement outside the recorded keyword-auction structure, or incomplete export. Only platform information can tell which.

---

## 6. Multiplicity audit

25 tests recorded in the original pipeline were adjusted by Benjamini–Hochberg, Holm and Bonferroni. Eight of the 25 unadjusted p-values are below .05; **after adjustment none is below .05** under any of the three methods (smallest BH-adjusted p = .058). A sensitivity check over the rounding intervals of the stored p-values gave no BH-significant test under any rounding. The full 25-test table and the ten smallest unadjusted p-values are in the manuscript's Appendix B and in `research_wide_audit/`.

A historical robustness battery (a 47-specification curve over three realised outcomes, temporal-split and panel variants, regression-discontinuity and policy-change screens) was **not re-run on the final panel and carries no evidential weight** in the manuscript. It is retained for traceability only (see Section 10).

---

## 7. Design proposals for audit and disclosure

The manuscript derives three proposals for the design of audit and disclosure arrangements. They concern design, not sanctions; they are offered for discussion and validation and have not been tested as standards.

| Proposal | Content |
|---|---|
| **P-1** | Document serving structure before auditing, and place that task on the platform (pricing arrangement per campaign type; whether size enters the price rule; how campaigns link to keyword-auction records). |
| **P-2** | Report audit statistics per segment, state every analytic choice (outcome definition, arithmetic link, weighting, size measure, inference method, unconditional and conditional estimates), and apply a minimum-sample rule that reports small segments as "insufficient sample". |
| **P-3** | Grade the response to the strength of evidence and use audit statistics for screening, not proof. An illustrative A–D grading framework and a segment-level statement template are given in the manuscript (Tables 9–10); they are a discussion device, not a validated standard. |

---

## 8. What is and is not robust

**Robust to the checks run:** the unconditional size–CPC association is near zero; the spend-conditioned coefficient mirrors the size–clicks coefficient; its point estimate is negative in every specification; the website point estimate is negative under both weightings; no pooled or segment estimate is shown to be equivalent to zero within ±30%; linkage to keyword-auction records is near zero for two campaign types.

**Not robust:** 5% significance of the spend-conditioned association (depends on weighting, deletion and size measure); any price-discrimination reading of the negative sign; the positive shopping sign (changes under equal weights); heterogeneity across types (depends on inference method, fails multiplicity adjustment); any localisation of heterogeneity to a pair of segments; any causal reading of the shift when local-business-exposed advertisers are excluded (that shift, from −0.253 to −0.499, shows sensitivity to advertiser composition, not an effect of local-business campaigns on prices).

---

## 9. Limitations

1. **One platform, one window.** Whether the pattern recurs elsewhere is a question for replication.
2. **Associational.** No causal identification; spend may itself partly reflect size-related conditions.
3. **CPC is not a price for comparable placements.** The statistic cannot separate a lower price per auction from higher click yield caused by quality, targeting, keyword mix, bidding strategy or placement.
4. **Selection on click days.** CPC is undefined on days without clicks; if click incidence depends on size, the sample is selected on an outcome-related event.
5. **Unbalanced panel.** 122 of 228 advertisers contribute one observation; the two weightings define different estimands.
6. **Small segments.** The 27- and 17-advertiser segments have wide minimum detectable effects and fewer than 30 clusters.
7. **Linkage is an availability indicator.** It cannot distinguish a different serving arrangement from incomplete export.
8. **Conditions S and R were formulated after seeing results.** They are practical conditions from one case, not tested propositions.
9. **Outcomes excluded by design.** Conversion and ROAS are excluded under Condition M; the statistic is untestable there, not violated.

---

## 10. Repository structure

The directory tree reflects the project's full history. **Only part of it supports the manuscript's evidence base**, and the annotations below say which part. Directories marked *legacy* are kept for traceability and are not cited as support for any claim in the manuscript.

```
structural-signal-irrelevance/
├── README.md
├── FUTURE_RESEARCH_STUDY2.md                 # legacy: descoped companion study (not part of the manuscript)
├── FUTURE_RESEARCH_STUDY3.md                 # forward-looking: pre-specified follow-up ideas
├── LICENSE
├── requirements.txt
├── config/
│   └── config.yaml                           # paths, fixed seeds, bootstrap settings
├── data/
│   └── README.md                             # schema description and access instructions (raw data not redistributed)
├── src/
│   ├── utils/
│   │   ├── io.py
│   │   └── identifiers.py
│   └── pipeline_v4/
│       ├── step0_data_prep_v4.py             # table linkage, sample construction, advertiser-day panel
│       ├── step1_variance_decomposition_v4.py# preliminary multilevel variance structure
│       ├── step2_advertiser_size_fairness_v4.py  # pooled and segment size models (SSI statistic)
│       ├── step3_churn_appendix_v4.py        # legacy: unrelated churn-prediction appendix
│       └── step4_synthesis_v4.py             # assembles tables and summary outputs
├── supplementary_robustness/                 # Tier 2 sensitivity checks (weighting, size measure, alternative outcomes)
│   ├── supplementary_robustness_README.md
│   ├── 01_alternative_outcome_mediation.md / .py
│   ├── 02_boundary_conditions.md / .py
│   └── 03_equivalence_and_sensitivity_notes.md / .py
├── supplementary_identification/             # legacy screening: RDD / policy-change designs, not re-run on the final panel
│   ├── SCREENING_SUMMARY.md
│   ├── step11_alt_identification_RDD_policy.py
│   ├── step11b_donut_hole_full_scan.py
│   └── step11c_customer_level_reanalysis.py
├── supplementary_localbiz_exploratory/       # local-business exposure and composition sensitivity (Tier 2 / Tier 3)
│   ├── README.md
│   └── localbiz_core_analysis.py
├── supplementary_mitigation_study/           # legacy: exploratory model-input mitigation study, not part of the manuscript
│   ├── README.md
│   ├── mitigation_common.py
│   ├── step_m0_pregate.py
│   ├── step_m1_exploratory_scan.py
│   ├── step_m2_m3_model_class_bootstrap.py
│   └── campaign_stratified_confirm/
│       ├── rq3_confirm_v2_campaign_stratified_full.py
│       ├── rq3_confirm_v2_patch_ols_stratified.py
│       ├── rq3_confirm_v2_robust_summary_postprocess.py
│       └── README.md                         # entry B10/B11 summary + run order
├── research_wide_audit/                      # 25-test multiplicity audit (BH, Holm, Bonferroni)
│   ├── README.md
│   └── research_wide_audit_core.py
├── figures/
│   ├── Figure1_variance_decomposition.png
│   ├── Figure2_fairness_forest_plot.png
│   ├── Figure3_specification_curve_placebo.png        # legacy battery; no evidential weight
│   ├── Figure4_churn_benchmark.png                    # legacy: unrelated appendix
│   ├── Figure5_coldstart_funnel_and_RQ1_null.png      # legacy: companion study
│   ├── Figure6_RQ2_horizon_RQ3_lift.png               # legacy: companion study
│   ├── Figure7_mediation_forest.png
│   ├── Figure8_boundary_condition_forest.png          # segment estimates by campaign type
│   ├── Figure9_tost_equivalence.png                   # legacy: companion study
│   ├── Figure10_integrated_framework.png              # legacy: spans the companion study
│   ├── Figure11_identification_screening.png          # legacy screening
│   ├── Figure12_h3_leave_one_type_out.png             # legacy filename ("h3"); composition sensitivity
│   ├── Figure13_serving_structure.png                 # keyword-record linkage by campaign type
│   ├── Figure14_multiplicity_audit.png
│   ├── Figure15_combinatoric_null_model.png
│   ├── Figure16_mitigation_model_class_bootstrap.png  # legacy: mitigation study
│   ├── Figure17_strategy_model_landscape.png          # legacy: mitigation study
│   ├── Figure18_mitigation_evidentiary_process.png    # legacy: mitigation study
│   ├── Figure19_campaign_stratified_model_class_bootstrap.png  # legacy: mitigation study
│   └── scripts/
│       └── figureN_*.py                      # one generation script per figure
├── appendix/
│   ├── churn_prediction_rq4.md               # legacy: unrelated appendix
│   ├── exploratory_industry_classification.md
│   └── hypothesis_id_legacy_mapping.md       # maps legacy H1/H2/H3/RQ labels to current terminology
├── docs/
│   ├── METHODOLOGY_NOTES.md                  # dated log of analytic decisions and corrections
│   ├── RESULTS_SUMMARY.md                    # full statistical tables
│   └── DESIGN_ARTIFACT.md                    # redirect stub (legacy)
└── manuscript_prep/
    ├── literature_anchors.md                 # working literature list
    └── paperization_notes.md                 # open framing threads
```

**Legacy terminology.** Earlier drafts used hypothesis labels (H1, H2, "H3") and research-question labels (RQ2a–c, M1–M3) that do not appear in the manuscript. Some script variables, figure titles and file names still carry them; `appendix/hypothesis_id_legacy_mapping.md` documents the mapping. The manuscript's own structure is: two research questions (RQ1 on interpretation and stability of the spend-conditioned coefficient; RQ2 on serving structure and its limits for segment-level audit), three evidentiary tiers, and three design proposals.

---

## 11. Manuscript-to-repository map

| Manuscript element | Where it lives |
|---|---|
| Sample flow (Appendix A) | `src/pipeline_v4/step0_data_prep_v4.py`, `docs/RESULTS_SUMMARY.md` |
| Pooled model and segment models (Tables 3, 6) | `src/pipeline_v4/step2_advertiser_size_fairness_v4.py` |
| Role of spend conditioning, clicks check (Table 4) | `supplementary_robustness/` and the independent re-implementation described in Section 4 |
| Stability checks: weighting, deletion, size measure (Table 5) | `supplementary_robustness/` |
| Keyword-record linkage (Table 7, Figure 3) | `src/pipeline_v4/step0_data_prep_v4.py`; closest figure file `figures/Figure13_serving_structure.png` |
| Segment forest plot (Figure 2) | closest figure file `figures/Figure8_boundary_condition_forest.png` |
| Multiplicity audit (Appendix B) | `research_wide_audit/` |
| Local-business exposure and spend-share interactions (Appendix C) | `supplementary_localbiz_exploratory/` |
| Audit workflow (Figure 1), grading framework and statement template (Tables 9–10) | Manuscript only; no code |

Figure files keep legacy names for continuity; regenerate with the scripts in `figures/scripts/` to obtain versions matching the manuscript.

---

## 12. How to reproduce

1. **Get the data or use the synthetic dataset.** The raw data are proprietary. `data/README.md` describes the schema; a schema-matched synthetic replication dataset is sufficient to run the full pipeline end-to-end.
2. **Configure.** Edit `config/config.yaml` for paths. Random seeds are fixed (seed 2026 for the wild cluster bootstraps, with 2,000 draws), so reruns on the same inputs reproduce the reported resampling results.
3. **Install.** `pip install -r requirements.txt`.
4. **Run the primary pipeline in order:** `step0_data_prep_v4.py` → `step1_variance_decomposition_v4.py` → `step2_advertiser_size_fairness_v4.py` → `step4_synthesis_v4.py` (all under `src/pipeline_v4/`).
5. **Run the Tier 2 and Tier 3 modules:** `supplementary_robustness/`, then `supplementary_localbiz_exploratory/localbiz_core_analysis.py`.
6. **Run the multiplicity audit:** `research_wide_audit/research_wide_audit_core.py`.
7. **Regenerate figures** with the matching script in `figures/scripts/`.

Directories marked *legacy* in Section 10 are not needed to reproduce the manuscript's results. Results from the synthetic dataset will not match the manuscript's numbers; they verify that the pipeline runs, not the empirical findings.

---

## 13. Data availability, disclosures and citation

**Data availability.** The raw data are proprietary to the data provider and not publicly redistributable; access requests may be directed to the corresponding author named on the manuscript's title page. This repository releases the complete analysis code, fixed random seeds for all stochastic procedures, and a schema-matched synthetic replication dataset.

**Disclosure.** One author is employed by the data provider (named on the manuscript's title page); the other authors have no financial or employment relationship with the data provider. The manuscript states that study design, analysis and interpretation were conducted independently of the data provider. The statistic and tiers were fixed in the original pipeline before the sensitivity analyses, and every post hoc analysis that was run is reported.

**Generative AI.** The authors used a generative AI assistant (Claude, Anthropic) during manuscript preparation, as declared in the manuscript. The authors reviewed and edited the output and take full responsibility for the content.

**License.** See `LICENSE`.
