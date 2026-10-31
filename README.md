# Observable but not readable: what outside audits can say about how platforms treat small advertisers in South Korea

> **Status and how to read this repository.** This is a research repository documenting the analysis pipeline behind the manuscript. It is not a publication, and nothing here should be cited as a peer-reviewed result. Where wording differs, **the manuscript is the authoritative statement of results**. The repository covers one platform, one observation window and one linear conditional statistic. It makes **no fairness, discrimination or causal claim**: every coefficient is a conditional association, and the manuscript's central point is what such a statistic can and cannot support when the serving structure is only partly observable.

---

## Table of contents

1. [At a glance](#1-at-a-glance)
2. [Argument and research questions](#2-argument-and-research-questions)
3. [The statistic and the four conditions](#3-the-statistic-and-the-four-conditions)
4. [Data and sample flow](#4-data-and-sample-flow)
5. [Analysis status and inference](#5-analysis-status-and-inference)
6. [Results](#6-results)
7. [Design considerations for audit and disclosure](#7-design-considerations-for-audit-and-disclosure)
8. [What is and is not robust](#8-what-is-and-is-not-robust)
9. [Limitations](#9-limitations)
10. [Repository structure](#10-repository-structure)
11. [Manuscript-to-repository map](#11-manuscript-to-repository-map)
12. [How to reproduce](#12-how-to-reproduce)
13. [Data availability, disclosures and citation](#13-data-availability-disclosures-and-citation)
14. [Open reconciliation items](#14-open-reconciliation-items)

---

## 1. At a glance

| | |
|---|---|
| **Setting** | Commerce search-advertising platform in South Korea (Naver); data from an advertising agency that manages sponsored-search accounts |
| **Window** | 23 August 2025 to 21 July 2026 |
| **Unit** | Advertiser-day, restricted to days with at least one click so that cost per click (CPC) is defined |
| **Panel** | 15,361 advertiser-days, 228 advertisers (highly unbalanced: 122 advertisers contribute a single day; maximum 193) |
| **Outcome** | log CPC (daily billed cost divided by daily clicks) |
| **Conditioning variable** | Spend: standardised log(1 + daily billed cost) |
| **Size measure** | Standardised log(1 + all-time number of ad groups); constant within advertiser, so identified between advertisers |
| **Statistic** | c′: the coefficient on size in a linear regression of log CPC on spend and size, with inference clustered by advertiser |

**Main findings (all conditional associations; coefficients refer to one standard deviation of size).**

| Finding | Evidence | Reading |
|---|---|---|
| Serving-structure information differs sharply by campaign type | Keyword-auction records link to 96.8% / 94.8% / 92.9% of ad groups (website / power content / brand-new product) but to 0.0% / 0.7% (local business / shopping) | Three campaign types can be audited; two have undetermined applicability |
| The spend-conditioned statistic is a cost-efficiency measure, not a price | c′ = −0.253 (SE 0.135; 95% CI −0.517 to 0.012; p = .062; wild cluster bootstrap p = .072–.074); clicks-model mirror coefficient +0.255 | By construction (log CPC = log cost − log clicks); at equal spend about 22% lower CPC, equivalently about 29% more clicks |
| Without conditioning on spend the association is bounded but not shown to be negligible | c0 = 0.000 (SE 0.139; 95% CI −0.271 to 0.272; p = .998) | Excludes size-related gaps larger than about 24% lower or 31% higher CPC per SD |
| Significance of c′ depends on declared analytic choices | p = .062 (primary), < .001 (equal-advertiser weights), .001 (campaign count), .067 (keyword count); p < .05 in 15.7–25.6% of random deletions of 5–15 advertisers | Weighting and size measure must be declared |
| Segment pattern is a descriptive lead only | Joint Wald p = .023, but wild cluster bootstrap p ≈ .22–.26 and BH-adjusted p = .081 | Does not survive resampling or multiplicity adjustment |
| Equivalence not established | Two one-sided tests (TOST) at ±10% / ±20% / ±30% give p = .877 / .698 / .471; pooled MDE ≈ 0.38 log points | See Section 6.1 and Section 14 |

**One-line summary.** Observable outcomes are not automatically readable: the computable statistic measures click efficiency rather than price, its significance moves with defensible analytic choices, and the two segments with positive point estimates link to almost no keyword-auction records and are too small to audit. Whether any segment is disadvantaged remains open, and the data identify no pricing mechanism.

---

## 2. Argument and research questions

**Observability versus readability.** Audit and disclosure regimes (for example the EU Platform-to-Business Regulation, Digital Services Act and Digital Markets Act, and the Korean bill introduced in December 2025) assume that once platform outcomes are observable, someone outside the platform can read them as evidence of how business users are treated. The manuscript separates two things:

- **Observability**: whether traces of platform behaviour can be seen by someone other than the operator.
- **Readability** (the concept added here): *the extent to which a statistic computed from outcome records can bear the interpretation that a regulatory question requires.* It is a property of the relation between a statistic and the question put to it, not of the system's disclosure.

**Three levels kept apart throughout.**

1. **Observable statistic.** The coefficient on size in the regression of log CPC on spend and size. A non-significant coefficient does not establish conditional independence, because nonlinear relations are not detected and power may be low.
2. **Auditability.** Whether the statistic can be computed and read reliably for a segment.
3. **Fairness or discrimination judgement.** This requires causal, institutional and intent evidence that no observational statistic supplies. The repository makes no such claim in either direction.

**Research questions** (descriptive; not tested against pre-specified predictions).

| RQ | Question |
|---|---|
| **RQ1** | What can an outside auditor establish about the association between advertiser size and CPC, net of spend? **(a)** How large a size-related CPC gap can be bounded without conditioning on spend? **(b)** What does the spend-conditioned association measure? **(c)** Under which declared choices of weighting, inference, sample composition and size measure does it change? |
| **RQ2** | What do the available records reveal about serving structure by campaign type? **(a)** For which campaign types is serving-structure information available? **(b)** At what scale does segment-level audit become informative? |

---

## 3. The statistic and the four conditions

**Notation.** S = advertiser size (structural attribute), B = spend (behavioural signal), X = other covariates, Y = outcome. Equal treatment in the individual-fairness sense would require Y ⊥ S | (B, X), called *structural signal irrelevance*. The statistic c′ is a linear, observational probe of that condition and nothing more.

**Models** (manuscript equations 2–5).

```
log CPC_it = α  + b·SPEND_it + c′·SIZE_i + ε_it        (2) primary, spend-conditioned
log CPC_it = α0 + c0·SIZE_i  + u_it                    (3) unconditional comparator †
c0 = c′ + b·δ   (δ = slope of spend on size)           (4) omitted-variable identity
log CPC_it = α_s + b_s·SPEND_it + c_s·SIZE_i + ε_it    (5) segment model; H0: c_website = c_local = c_shopping
```

**What the statistic measures.** Because CPC is billed cost divided by clicks, log CPC = log cost − log clicks holds by construction (equation 6). When spend is the conditioning variable, the size coefficient in the CPC regression is therefore, up to the approximation arising from log(1 + cost), the negative of the size coefficient in a regression of log clicks on the same regressors. With the exact log of cost the two sum to zero to numerical precision. The statistic is exactly a cost-efficiency measure; it cannot separate a lower price per auction from a higher click yield caused by quality, targeting, keyword mix or bidding strategy. The spend coefficient (b = 1.277) is not interpreted. The implied size–spend slope δ is about 0.20 (derived, not separately estimated): larger advertisers spend more per day.

**Four conditions for reading such a statistic.** Each can fail separately. They were formulated from this case *after* seeing the results; they are an organising device and candidates for validation on other platforms, not tested propositions, and they are not used to exclude any segment.

| Condition | Meaning | Where it stands in these data |
|---|---|---|
| **Measurement** | Behavioural signal and outcome are measured completely, independently of the attribute under study, and without an arithmetic link that changes what the coefficient means | Fails for a price reading: CPC and spend are arithmetically linked; the statistic is a click-efficiency measure |
| **Structure** | The statistic presupposes an auction- or bidding-based pricing arrangement for the segment concerned | Assessable for three campaign types; undetermined for local business and shopping |
| **Stability** | The statistic is stable to reasonable changes in weighting, inference method, sample composition and size measure | Mapped: sign stable, significance not |
| **Sufficiency** | The segment is large enough to discriminate between substantively different states of the world | Quantified: MDE80 about 50% (website), 66% (shopping), 107% (local business) |

*(Earlier repository drafts used three conditions labelled M, S and R. R is now split into Stability and Sufficiency, and the labels are dropped because S clashes with the manuscript's use of S for size.)*

---

## 4. Data and sample flow

Four linked tables are used: an ad-performance log (19,373,916 records; 321 advertisers), a campaign table, an ad-group table and a keyword table. Campaign types are numeric codes in the export; labels were assigned from campaign-name content. The export contains no description of the pricing arrangement applicable to each campaign type. **The data cannot separate a platform-side serving arrangement from incomplete export of records**, which shapes interpretation throughout and is itself one of the findings.

The data are agency-managed accounts, a convenience (non-probability) sample: inclusion probabilities are unknown and no population benchmark exists. Findings are a case study of what an outside audit can establish (analytic, not statistical, generalisation).

| Stage | n | Note |
|---|---|---|
| Advertisers in the performance log | 321 | 19,373,916 records |
| No ad-group metadata (no size measure; excluded) | 58 | Outside the analysis |
| Advertisers in the ad-group metadata table | 263 | 9,823 ad-group rows |
| In metadata but not in the analysis frame | 2 | Positive cost (3,290 and 36,870 billing units); at most 19 advertiser-days (about 0.1% of the panel); reason for absence not established |
| Identified advertisers in the analysis frame | 261 | 5,526 rows, of which 4,407 carry an advertiser identifier; 1,119 cannot be attributed |
| Zero cumulative cost (excluded) | 33 | |
| **Core sample** (positive cumulative cost) | **228** | Smallest included cumulative cost: 50 billing units |
| **Advertiser-day panel** | **15,361** | Days with at least one click; median 1, maximum 193 per advertiser; 75th percentile 173; top 20 contributors account for 24.8% |

**Segments** (plurality-spend campaign type): website = 184 advertisers (11,894 advertiser-days); local business = 27 (1,306); shopping = 17 (2,161). No core advertiser has power content or brand/new product as its plurality category. Core advertisers with any spend by type: website 202, power content 13, brand/new product 0, local business 72, shopping 24. "Local-business exposure" means any non-zero local-business spend (72 of 228 advertisers).

**Weighting.** Primary estimates weight each advertiser-day equally (w = 1), a choice fixed before the sensitivity analyses. Equal-advertiser weights (w = 1/nᵢ, where nᵢ is the number of observed days of advertiser i) define a different estimand and are reported as an exploratory alternative (†). Neither is the better measure of equal treatment.

**Selection note.** Restricting to days with at least one click means the estimand concerns CPC on days with clicks; if click incidence depends on size, the sample is selected on an outcome-related event.

---

## 5. Analysis status and inference

The manuscript distinguishes **analyses fixed before the sensitivity work** from **analyses prompted by results**, which are labelled exploratory and marked † in the manuscript. The analysis was not pre-registered, and every post hoc analysis relevant to the research questions is reported. The repository's internal tiers map onto this as follows.

| Repository tier | Manuscript status | Contents |
|---|---|---|
| **Tier 1** | Fixed before sensitivity work | Pooled model, segment stratification, joint heterogeneity test, 25-test multiplicity audit |
| **Tier 2** | Exploratory (†) | Unconditional comparator and clicks-identity check, wild cluster bootstrap, equal-advertiser weights, influence and random-deletion comparisons, local-business-exposure exclusion, alternative size measures, equivalence tests, minimum detectable effects |
| **Tier 3** | Exploratory (†); not interpreted | Spend-share interactions (including the power-content share × size term, which rests on 13 advertisers with no plurality in that type) |

The tiers are a repository organising device and do not appear in the manuscript. Keyword-record linkage (RQ2a) is a descriptive result reported as such; it is not a Tier 3 item. No downstream tier upgrades an upstream one.

**Inference.** Because size varies only between advertisers, inference is clustered by advertiser with a small-sample correction (the primary p-values). Because two segments have few clusters and many clusters contribute one observation, **wild cluster bootstrap p-values (†)** are also reported, with Rademacher and Webb weights, 2,000 draws and fixed seed 2026. Where the two disagree (for example the heterogeneity test), both are reported and the disagreement is treated as a finding about fragility. Minimum detectable effects (MDE) at 80% power are 2.81 × SE. Equivalence is assessed by TOST with margins of ±10%, ±20% and ±30% on CPC. Across the 25 tests recorded in the original pipeline (which include boundary, placebo and discontinuity screens), Benjamini–Hochberg (BH), Holm and Bonferroni corrections are applied. A difference between a significant and a non-significant result is not itself read as significant.

**Replication note.** All estimates come from the original pipeline except the clicks-identity check (manuscript Figure 3a), which uses an independent re-implementation on the final panel. It reproduced the primary coefficient (−0.255 against −0.253) and the equal-weight coefficient (−0.359 against −0.356); differences reflect the standardisation convention and bootstrap draws, and no conclusion depends on them. Tables below state which source each number comes from.

---

## 6. Results

### 6.1 Pooled association, with and without spend (RQ1a, RQ1b)

| Model | Weights | Coefficient | 95% CI | p | Wild bootstrap p | Source |
|---|---|---|---|---|---|---|
| log CPC ~ size (unconditional, c0) † | Advertiser-day | 0.000 | [−0.271, 0.272] | .998 | .997 (Webb / Rademacher) | Re-implementation |
| log CPC ~ spend + size (c′, primary) | Advertiser-day | −0.253 | [−0.517, 0.012] | .062 | .072–.074 | Original pipeline |
| log CPC ~ spend + size (c′, check) † | Advertiser-day | −0.255 | [−0.522, 0.013] | .062 | .072–.073 | Re-implementation |
| log clicks ~ spend + size (mirror check) † | Advertiser-day | +0.255 | [−0.013, 0.522] | .062 | .073 | Re-implementation |
| log CPC ~ spend + size (c′) † | Equal advertiser | −0.356 | n/r | < .001 | < .001 | Original pipeline |
| log CPC ~ spend + size (c′, check) † | Equal advertiser | −0.359 | [−0.524, −0.193] | < .001 | < .001 | Re-implementation |
| log CPC ~ size (unconditional) † | Equal advertiser | +0.078 | [−0.098, 0.255] | .385 | .401 | Re-implementation; repository-only result, not in manuscript |

n/r = not reported.

- **Bound without spend.** The unconditional interval excludes size-related CPC gaps larger than about 24% lower or 31% higher per SD of size.
- **With spend held constant**, −0.253 corresponds to about 22% lower CPC, or equivalently about 29% more clicks, per SD of size.
- **Power and equivalence.** The pooled MDE † is about 0.38 log points (about 46% CPC difference per SD), which exceeds the point estimate. TOST † at ±10% / ±20% / ±30% gives p = .877 / .698 / .471 (these values reproduce from the spend-conditioned c′ and its SE), so equivalence is not established at any margin. The margins are not substantively derived; see Section 14.

### 6.2 A stability map (RQ1c)

The pooled point estimate is negative in every specification examined. All rows below are exploratory (†) except the primary one.

| Specification | Advertisers | c′ | p |
|---|---|---|---|
| Primary: ad-group count, advertiser-day weights | 228 | −0.253 | .062 (bootstrap .072–.074) |
| Equal-advertiser weights (different estimand) | 228 | −0.356 | < .001 |
| Control for number of campaign types | 228 | −0.235 | .076 |
| Campaign count as size (95% CI −0.523 to −0.130) | 228 | −0.326 | .001 |
| Keyword count as size | 221 | −0.241 | .067 |
| Excluding the 15 advertisers flagged by the original influence diagnostic | 213 | −0.398 | < .001 |
| Excluding the 72 advertisers with any local-business spend (bootstrap 95% interval of the shift −0.513 to −0.045) | 156 | −0.499 | not reported |

How to read the map:

1. **Weighting.** The equal-weight result is a different estimand, not a more reliable version of the primary one, and it carries uncertainty from the 122 single-day advertisers.
2. **Outcome-informed exclusions are not independent robustness evidence.** Recomputed influence diagnostics identify only 7 of the original 15 flagged advertisers. The informative comparison is random deletion: the mean estimate stays at −0.252 to −0.253, but p < .05 arises in 15.7%, 22.5% and 25.6% of 5,000 draws when 5, 10 or 15 advertisers are deleted. The primary result lies close to the significance boundary.
3. **Size measure.** Moving between ad-group, campaign and keyword counts moves the result across the 5% line (ad-group and campaign counts correlate .72; magnitudes are not comparable across measures).
4. **Composition check.** Random deletion of 72 advertisers matches or exceeds the local-business shift (−0.247) in 1.0% of draws and a size-matched placebo in 0.7%, yet excluding the ten most influential advertisers produces a shift of similar size (−0.21), and local-business exposure correlates .58 with the number of campaign types. The shift shows sensitivity to advertiser composition, **not** an effect of local-business campaigns on prices.
5. **Consistency of sign confirms no pricing mechanism**: it is equally what one would see if larger advertisers obtained more clicks per unit of spend for reasons unrelated to platform treatment.

### 6.3 Segments: a descriptive lead and the scale at which audit becomes informative (RQ2b)

| Segment | Advertisers | Advertiser-days | c′ (primary) | 95% CI | p | MDE80 (log points) | MDE80 (CPC difference) |
|---|---|---|---|---|---|---|---|
| Website | 184 | 11,894 | −0.279 | [−0.560, 0.003] | .052 | 0.404 | about 50% |
| Local business | 27 | 1,306 | +0.312 | [−0.177, 0.801] | .211 | 0.726 | about 107% |
| Shopping | 17 | 2,161 | +0.245 | [−0.089, 0.579] | .151 | 0.509 | about 66% |
| Pooled | 228 | 15,361 | −0.253 | [−0.517, 0.012] | .062 | about 0.38 | about 46% |

- Under equal-advertiser weights † the website estimate is −0.353 (negative under both weightings), the local-business estimate +0.210, and the shopping estimate **−0.194** (the shopping sign turns negative).
- **Heterogeneity.** The joint Wald test of equation (5) gives p = .023 (asymptotic). A wild cluster bootstrap † of the same statistic gives p ≈ .22–.26. Under equal advertiser weights † the Wald test gives p = .064 and the bootstrap p = .237 (Webb) and .246 (Rademacher). Within the 25-test audit the heterogeneity test has adjusted p-values of .081 (BH), .430 (Holm) and .566 (Bonferroni).
- **The segment pattern is a descriptive lead, not a finding**, and no difference is localised to a pair of segments. The two small segments could detect with 80% power only CPC differences of about 107% and 66% per SD.

### 6.4 Keyword-record linkage: serving-structure information (RQ2a)

Share of ad groups of each campaign type that link to keyword-auction records (equation 1). The partition does not depend on any threshold: shares are at most 0.7% for low-linkage types and at least 92.9% for high-linkage types.

| Campaign type | Ad groups linked to keyword-auction records | Core advertisers with plurality in type | Core advertisers with any spend in type |
|---|---|---|---|
| Website | 96.8% | 184 | 202 |
| Power content | 94.8% | 0 | 13 |
| Brand/new product | 92.9% | 0 | 0 |
| Local business | 0.0% | 27 | 72 |
| Shopping | 0.7% | 17 | 24 |

Linkage is computed over ad groups, not advertisers, so campaign types with no plurality advertisers can still show a share. Three statements are kept apart:

- **Observed fact:** linkage differs sharply by campaign type.
- **Possible interpretation:** the extent to which an outside auditor can verify the pricing mechanism differs by type.
- **Unconfirmed claim:** that serving structure itself differs and that this produces outcome differences. The data do not support this. A missing match may reflect a serving arrangement outside the recorded keyword-auction structure or incomplete export, and only the platform can tell which.

### 6.5 Multiplicity audit

Twenty-five tests recorded in the original pipeline (including boundary, placebo and discontinuity screens) were adjusted by BH, Holm and Bonferroni. Eight have unadjusted p < .05; **after adjustment none is below .05** under any method (smallest BH-adjusted p = .058). The pipeline stored some p-values rounded; the heterogeneity, power-content share and two placebo p-values were replaced with full-precision recomputations (placebo p-values from 5,000 draws, (k + 1)/(B + 1)), and a check over the remaining rounding intervals gave no BH-significant test under any rounding.

| Test | Unadjusted p | BH | Holm | Bonferroni |
|---|---|---|---|---|
| Power-content share × size (not interpreted) | .0043 | .058 | .109 | .109 |
| Full sample vs exclusion of local-business exposure | .0060 | .058 | .144 | .150 |
| Size-matched placebo (empirical) | .0070 | .058 | .161 | .175 |
| Heterogeneity, joint Wald | .0226 | .081 | .430 | .566 |
| Regression-discontinuity screen at log size 2.515 | .0480 | .150 | .864 | 1.000 |

The power-content term rests on 13 advertisers, none with power content as plurality category; inferential procedures disagreed (permutation p = .406), so it is not interpreted. The full 25-test table is in `research_wide_audit/`.

A historical robustness battery (a 47-specification curve over three realised outcomes, temporal-split and panel variants, and policy-change screens) was **not re-run on the final panel and carries no evidential weight** in the manuscript. It is retained for traceability only (Section 10).

### 6.6 Summary by research question

| RQ | What is established |
|---|---|
| 1a | Bounded: roughly 24% lower to 31% higher CPC per SD without spend; equivalence at ±10/20/30% not established for the reported tests |
| 1b | Established: the conditional statistic is a click-efficiency measure, not a price |
| 1c | Established: significance depends on weighting and size measure, so both must be declared; sign is stable |
| 2a | Established (descriptive): three auditable campaign types, two undetermined |
| 2b | Quantified: MDE80 about 50% / 66% / 107%; heterogeneity is a descriptive lead only |

Entries refer to the evidence in these data, not to the platform's treatment of advertisers.

---

## 7. Design considerations for audit and disclosure

The manuscript derives three design considerations (abbreviated **P-1 to P-3** in this repository). They concern the design of audit and disclosure arrangements, not sanctions; they are untested and offered for discussion. Each follows from a specific point where the statistic could not be read.

| Consideration | Content |
|---|---|
| **P-1** | Platform operators should document serving structure before audits begin: pricing arrangement per campaign type, whether advertiser attributes including size enter the price rule, and how campaigns link to keyword-auction records. The cost of becoming legible should fall on the platform, not on small advertisers. |
| **P-2** | Regulators and national authorities should ask for audit statistics per segment with every analytic choice declared: weighting, size measure, inference method, at least one alternative, whether the outcome is arithmetically tied to the conditioning variable, and the unconditional association beside the conditional one. Segments below a minimum power requirement fixed in advance could be reported as "insufficient sample" (no numerical threshold is proposed). |
| **P-3** | Regulators, mediation bodies and advertisers' agencies should grade the response to the strength of evidence and use audit statistics to **screen, not to prove**. |

**Illustrative evidence grading (manuscript Table 4; a discussion device, not a validated standard, and no grade is a finding of violation or of fairness).**

| Grade | Serving-structure condition | Evidence profile | Placement in this study |
|---|---|---|---|
| A | Assessable; arrangement confirmed by platform | Strong evidence robust to specification, inference, weighting and multiplicity, with separate causal support | None |
| B | Assessable (keyword-linked records available); arrangement to be confirmed | Association present or marginal; provisional | Website segment (−0.28; p = .052) |
| C | Undetermined (low linkage) or segment below minimum-sample rule | Direction may be observed but is not interpreted | Local-business and shopping segments |
| D | Assessable; arrangement confirmed by platform | Equivalence within a stated substantive margin; sensitivity checks do not contradict | None (not reached by any estimate here) |

A segment-level audit statement would minimally report: segment and period; serving structure; segment size and MDE; outcome, size measure and weighting with an alternative; estimate with direction, interval and p; heterogeneity; equivalence; grade; and limits. For the website segment: 184 advertisers, 11,894 advertiser-days, MDE80 0.40, estimate −0.28, equal-weight alternative −0.35, limits (CPC is a cost-efficiency measure; one platform), grade B.

**Policy relevance.** The Korean bill introduced in December 2025 would prohibit discriminatory treatment of user businesses without defining a comparator; the EU instruments (P2B Article 7 ranking-parameter description, DSA audit and researcher access, DMA price information) do not require outcomes by advertiser size or serving structure. The considerations above could extend existing duties rather than create new prohibitions.

---

## 8. What is and is not robust

**Robust to the checks run:** the unconditional size–CPC association has a point estimate of zero with a bounded interval; the spend-conditioned coefficient mirrors the size–clicks coefficient; the **pooled** point estimate is negative in every specification examined; the website point estimate is negative under both weightings; no pooled or segment estimate is shown to be equivalent to zero within the tested margins in the spend-conditioned analysis; linkage to keyword-auction records is near zero for two campaign types.

**Not robust:** 5% significance of the spend-conditioned association (depends on weighting, deletion and size measure); any price-discrimination reading of the negative sign; the positive shopping sign (changes under equal weights); heterogeneity across types (depends on inference method, fails multiplicity adjustment); any localisation of heterogeneity to a pair of segments; any causal reading of the shift when local-business-exposed advertisers are excluded (from −0.253 to −0.499).

---

## 9. Limitations

1. **One platform, one agency, one window.** Accounts may not represent all advertisers, search platforms change over time, and findings are time-bound. Whether the pattern recurs elsewhere is a question for replication.
2. **Associational.** Linear conditional associations support no causal, pricing-mechanism or fairness claim in either direction.
3. **CPC is not a price for comparable placements.** The statistic measures cost efficiency and cannot separate a lower price per auction from a higher click yield caused by quality, targeting, keyword mix, bidding strategy or placement. Auction-level prices and registered bids would be needed. Registered-bid analyses could not be reproduced on the final panel and are not reported.
4. **Size measurement and sample.** 20.2% of ad-group rows carry no advertiser identifier and 58 of 321 logged advertisers (18.1%) have no size measure, so measurement error (which would typically attenuate coefficients) and non-random exclusion are possible. Size is a cumulative ad-group count identified between advertisers and may post-date part of the window.
5. **Selection on click days.** CPC is undefined on days without clicks; if click incidence depends on size, the sample is selected on an outcome-related event.
6. **Unbalanced panel.** 122 of 228 advertisers contribute one observation; the two weightings define different estimands.
7. **Small segments.** The 27- and 17-advertiser segments have wide minimum detectable effects and few clusters.
8. **Linkage is an availability indicator.** It cannot distinguish a different serving arrangement from incomplete export.
9. **Unobserved advertiser behaviour.** Because accounts are agency-managed, bidding strategy, budget settings and keyword mix may differ with advertiser size and are not observed.
10. **Conditions and grading were formulated after seeing results** and have not been validated.
11. **Outcomes excluded by design (repository note).** Conversion and ROAS outcomes are not analysed because tracking backfill lag plausibly correlates with size; the statistic is untestable there, not violated.

Future work: platform-documented pricing arrangements, auction-level price and bid data, click-incidence analysis, substantively defined equivalence margins, longer windows and repeated periods, and pre-specified replication across platforms.

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

**Legacy terminology.** Earlier drafts used hypothesis labels (H1, H2, "H3") and research-question labels (RQ2a–c, M1–M3) that do not appear in the manuscript. Some script variables, figure titles and file names still carry them; `appendix/hypothesis_id_legacy_mapping.md` documents the mapping. The manuscript's own structure is: two research questions (RQ1a–c on interpretation and stability of the spend-conditioned coefficient; RQ2a–b on serving structure and the scale at which segment-level audit becomes informative), a split between analyses fixed in advance and exploratory analyses (†), and three design considerations. The repository's Tier 1/2/3 labels (Section 5) are an internal organising device only.

**Names that no longer match content.**

| File or label | Note |
|---|---|
| `01_alternative_outcome_mediation.*`, `Figure7_mediation_forest.png` | Legacy names. The manuscript reports no causal mediation and no alternative outcomes; its related content is the clicks-identity check, and every coefficient is a conditional association. |
| `step2_advertiser_size_fairness_v4.py` | "Fairness" is a legacy name; the manuscript makes no fairness claim. |
| `Figure12_h3_leave_one_type_out.png` | Legacy label; supports the composition-sensitivity analysis (Section 6.2). |
| `Figure2_fairness_forest_plot.png`, `Figure8_boundary_condition_forest.png` | Figure 8 is the closest file for the segment forest plot (manuscript Figure 2b). |

---

## 11. Manuscript-to-repository map

| Manuscript element | Where it lives |
|---|---|
| Table 1: sample flow | `src/pipeline_v4/step0_data_prep_v4.py`, `docs/RESULTS_SUMMARY.md` |
| Section 4.2: pooled and segment models (equations 2–5) | `src/pipeline_v4/step2_advertiser_size_fairness_v4.py` |
| Section 4.2: spend conditioning and clicks identity (Figure 3a) | `supplementary_robustness/` and the independent re-implementation described in Section 5 |
| Section 4.3: stability map, random deletion (Figure 3b, 3c) | `supplementary_robustness/`; no matching figure file, regenerate with `figures/scripts/` |
| Section 4.3: local-business-exposure and composition checks | `supplementary_localbiz_exploratory/`; closest figure `figures/Figure12_h3_leave_one_type_out.png` |
| Section 4.1: keyword-record linkage (Figure 2a) | `src/pipeline_v4/step0_data_prep_v4.py`; closest figure `figures/Figure13_serving_structure.png` |
| Section 4.4: segment estimates (Figure 2b) | `supplementary_robustness/02_boundary_conditions.py`; closest figure `figures/Figure8_boundary_condition_forest.png` |
| Section 4.5: multiplicity audit (Table 2) | `research_wide_audit/`; closest figure `figures/Figure14_multiplicity_audit.png`; full 25-test table in the anonymised repository |
| Figure 1: audit workflow; Table 4: grading and statement template | Manuscript only; no code |

Figure files keep legacy names for continuity; regenerate with the scripts in `figures/scripts/` to obtain versions matching the manuscript. Exact script-level mappings marked "closest" should be verified against the code.

---

## 12. How to reproduce

1. **Get the data or use the synthetic dataset.** The raw data are proprietary. `data/README.md` describes the schema; a schema-matched synthetic replication dataset is sufficient to run the full pipeline end-to-end.
2. **Configure.** Edit `config/config.yaml` for paths. Random seeds are fixed (seed 2026 for the wild cluster bootstraps, with 2,000 draws), so reruns on the same inputs reproduce the reported resampling results.
3. **Install.** `pip install -r requirements.txt`.
4. **Run the primary pipeline in order:** `step0_data_prep_v4.py` → `step1_variance_decomposition_v4.py` → `step2_advertiser_size_fairness_v4.py` → `step4_synthesis_v4.py` (all under `src/pipeline_v4/`).
5. **Run the exploratory modules:** `supplementary_robustness/`, then `supplementary_localbiz_exploratory/localbiz_core_analysis.py`.
6. **Run the multiplicity audit:** `research_wide_audit/research_wide_audit_core.py`.
7. **Regenerate figures** with the matching script in `figures/scripts/`.

Directories marked *legacy* in Section 10 are not needed to reproduce the manuscript's results. Results from the synthetic dataset will not match the manuscript's numbers; they verify that the pipeline runs, not the empirical findings.

---

## 13. Data availability, disclosures and citation

**Data availability.** The raw advertiser-level records are commercial data held by a third-party data provider (the advertising agency that supplied the accounts) and cannot be shared publicly; access may be arranged on reasonable request, subject to the provider's agreement. This repository releases the complete analysis code, fixed random seeds for all stochastic procedures, and a schema-matched synthetic replication dataset. An anonymised copy is available at <https://anonymous.4open.science/r/sponsored-search-size-audit-15AD/README.md>.

**Disclosure.** One author is employed by the data provider (named on the manuscript's title page); the other authors have no financial or employment relationship with the data provider. Study design, analysis and interpretation were conducted independently of the data provider. The pooled model, segment stratification, joint heterogeneity test and 25-test audit were fixed in the original pipeline before the sensitivity analyses; every post hoc analysis relevant to the research questions is reported in the manuscript.

**Generative AI.** The authors used a generative AI assistant (Claude, Anthropic) during manuscript preparation, as declared in the manuscript. The authors reviewed and edited the output and take full responsibility for the content.

**Citation.** Cite the manuscript, not this repository, once published. Until then, cite this repository only as unpublished replication materials.

**License.** See `LICENSE`.

---

## 14. Open reconciliation items

Items to settle between the manuscript and this repository before release. They do not change the structure in Section 10.

1. **Target of the equivalence tests.** The reported TOST p-values (.877 / .698 / .471) reproduce from the spend-conditioned c′ (−0.253, SE 0.135). The manuscript's Table 3 places them beside the unconditional c0, and the abstract and conclusion do not name the coefficient. Name the target explicitly and, if desired, report TOST for c0 separately (an exploratory analysis; the unconditional SE of 0.139 implies a narrower 90% interval that may fit inside ±30%). Margins are not substantively derived.
2. **Segment confidence intervals.** Segment CIs and MDEs are on the normal (z) scale; with 27 and 17 clusters, t(G−1) intervals would be wider. State which is used and, if changed, recompute.
3. **RD screen and legacy directories.** The regression-discontinuity screen (p = .048) is one of the 25 tests, while `supplementary_identification/` is marked as not re-run on the final panel. Record in `docs/METHODOLOGY_NOTES.md` which panel produced each of the 25 p-values.
4. **Registered bids.** README Limitation 3 states registered-bid analyses were not reproducible on the final panel; confirm the manuscript's statement that price-relevant variables are held only by the platform is worded to match (for example, "auction-level prices for comparable placements").
5. **Unreported analyses.** List in `docs/METHODOLOGY_NOTES.md` the analyses that were run but not reported (47-specification curve, RDD/policy screens, mitigation study, churn appendix, top-5/10/15 DFBETA exclusions, equal-weight unconditional estimate) with the reason (different question, or not re-run on the final panel).
6. **Disclosure.** Ensure the author–data-provider relationship stated in Section 13 also appears in the manuscript.
7. **Estimate sources.** Where a table mixes original-pipeline and re-implementation values, keep the "Source" column and the one-line standardisation note in the manuscript.
