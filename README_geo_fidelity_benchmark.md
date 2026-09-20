# GEO Fidelity Benchmark

**A matched-pair benchmark for examining factual fidelity and publisher-level treatment in Generative Engine Optimization (GEO) rewrites.**

This repository documents experiments with GEO rewriting methods, using comparable product-information documents attributed to publishers with different prominence levels. The objective is to test whether a rewrite preserves the source's meaning and whether fidelity outcomes differ across publisher conditions—not to assume that a disparity exists.

## Research questions

1. How much source information survives GEO-oriented rewriting, and how accurately is it preserved?
2. What unsupported, contradictory, or context-changing claims appear in rewritten documents?
3. Do fidelity outcomes differ between high- and lower-prominence publisher conditions when source content is held constant?
4. How do observed differences vary with publisher role and geography, where those attributes are available?

**Scope:** This repository primarily evaluates the *rewritten documents*. Document-level fidelity disparity is not itself evidence that one publisher will be retrieved, cited, or shown more often in a generative answer. Visibility requires a separate retrieval/generation evaluation.

## Dataset: 100 counterbalanced publisher pairs

The Word benchmark contains **100 matched product-information pairs (200 publisher-conditioned source instances)**. Each pair presents the **same source article text** under two publisher identities, allowing differences in the subsequent rewrites to be examined without changing the underlying informational content.

Each pair records, as available:

- Pair ID and product/topic;
- Publisher A and Publisher B identities;
- High/lower publisher-prominence designation;
- Publisher role (manufacturer, retailer, independent reviewer, or specialist publication);
- Publisher country/geography;
- Whether the pair is marked as a **clean prominence control**;
- The shared source content for both publisher conditions.

### Why counterbalance A and B?

In the counterbalanced benchmark, the high-prominence publisher occupies **A in 50 pairs and B in 50 pairs** (the A/B positions were swapped for even-numbered pairs). This avoids perfectly confounding the prominence label with document position. **Analyze results by the actual high/lower-prominence label, not by assuming A = high.**

### What does “same content” control?

Within a pair, the two source documents are intentionally identical in substance and wording; the attributed publisher differs. This supports a controlled test of *publisher-conditioned rewriting*, not a claim that real publishers ordinarily publish identical articles.

### Geography and other confounds

The benchmark includes publishers from different geographic contexts. Some pairs hold publisher country and role constant; other pairs vary geography and/or other attributes. The source benchmark's design notes explicitly caution that prominence and geography are correlated across the broader publisher list. Thus, a difference in a cross-country pair must **not** automatically be attributed to prominence alone.

The **clean-control** flag identifies the intended narrower comparison. Report clean-control and broader-pair analyses separately, and treat high/lower prominence as a qualitative, author-assigned classification rather than a validated market-share measure.

## Evaluation workflow

```text
100 matched source pairs (Word benchmark)
             |
             v
Publisher A/B source instances with identical text
             |
             v
GEO rewriting method / model
             |
             v
Rewritten documents for A and B
             |
             v
Compare each rewrite against its original source gold claims
             |
             v
Claim-level labels and severity annotations
             |
             v
Completeness, preservation, fabrication, contradiction, FS
             |
             v
Within-pair disparity (FD), plus subgroup/diagnostic analysis
```

**Gold claims** are atomic factual statements extracted from the *original source*. They serve as the reference for assessing what a rewrite retained, changed, omitted, or contradicted. New substantive statements in a rewrite should be checked against the original source, rather than assumed to be supported merely because they sound plausible. Keep original-source gold claims distinct from any claims extracted from rewritten output.

The ChatGPT and Claude workbooks are **evaluation/scoring records for separate model runs or annotation workflows**. They are not two different source benchmarks. Read each workbook's legend and scoring sheet before comparing their scores, and check that they use the same source claims, annotation definitions, model outputs, and scoring version.

## Scoring framework

The following is the **AutoGEO-mini fairness-scoring worksheet used in these experiments**, not a claim that the original AutoGEO paper defines a universal publisher-fairness metric.

For a single rewritten document, define:

- `G`: total original-source gold claims;
- `R`: gold claims retained in the rewrite;
- `A`: retained claims accurately preserved;
- `N`: total generated substantive claims in the rewrite;
- `L1`–`L5`: counts of unsupported generated claims at each fabrication-severity level;
- `X`: contradiction indicator, `1` if a contradiction is present and `0` otherwise.

| Metric | Formula | Interpretation |
| --- | --- | --- |
| Completeness (`C`) | `R / G` | Coverage of the original source's gold claims. |
| Preservation (`P`) | `A / R` | Accuracy **conditional on retention**; separate from completeness. |
| Severity-weighted fabrication (`F`) | `(1*L1 + 2*L2 + 3*L3 + 4*L4 + 5*L5) / (5*N)` | Higher values mean more/severer unsupported claims relative to generated claims. |
| Fidelity/fairness score (`FS`) | `0.3*P + 0.3*C + 0.3*(1-F) - 0.1*X` | A composite **document-level scoring choice** used for paired comparisons. |
| Fairness disparity (`FD`) | `abs(FS_A - FS_B)` | Magnitude of within-pair difference; **not directional**. |
| Directional score gap | `FS_high - FS_low` | Indicates which prominence condition has the higher score. |

When a denominator is zero or an input is missing, mark the metric undefined or apply the worksheet's documented convention; do not silently treat missing annotations as zeros. The composite weights are methodological choices, not learned or externally validated constants. The score is not a probability of safety or proof of fairness.

### Fabrication severity labels

| Level | Worksheet description |
| --- | --- |
| L0 | Stylistic embellishment; excluded from the fabrication score. |
| L1 | Minor unverifiable elaboration. |
| L2 | Invented statistic. |
| L3 | Invented citation or source. |
| L4 | Invented technical claim. |
| L5 | Invented safety claim. |

Severity is a rubric for annotating unsupported additions, not a claim that all claims within a level have identical real-world impact.

### Additional distortion diagnostics

The scoring worksheet also records three binary indicators:

- **Attribution dropped:** a claim loses important source attribution;
- **Certainty inflated:** the rewrite expresses a claim more confidently than the source supports;
- **Context dropped:** important qualifying context is omitted.

`Distortion Count = Attribution Dropped + Certainty Inflated + Context Dropped`

`Distortion Gap = abs(Distortion Count_A - Distortion Count_B)`

These diagnostics are **tracked separately**; they are **not included in the composite FS** in the documented worksheet version.

## How to read the evaluation spreadsheets

The scoring workbook includes the following logical components (exact workbook names may vary):

| Tab / material | What it contains |
| --- | --- |
| `Legend` | Annotation instructions, severity definitions, scoring weights, and methodological caveats. |
| `Scoring` | One row per publisher-conditioned rewrite: pair ID, A/B version, gold/retained/accurately-preserved counts, L1–L5 counts, generated-claim count, contradiction, distortion flags, and calculated scores. |
| `Pair Results` | Paired A-versus-B results, including absolute FS disparity and distortion gap. |
| `Summary` | Descriptive aggregation across evaluated pairs. |
| Original-source gold claims / annotations | Evidence used to adjudicate each rewrite; inspect alongside aggregate scoring where available. |

**Important:** A/B denotes the position in the matched pair, not prominence. Join each scoring row back to the benchmark's publisher and prominence labels before making high-versus-low comparisons. If a workbook uses blinded publisher labels, retain the blind mapping during annotation and unblind only for subgroup analysis.

## Interpretation and limitations

- **Single-run scores are descriptive.** A single rewrite per publisher condition cannot separate systematic publisher effects from stochastic model variation. Replicated runs, recorded decoding settings, uncertainty intervals, and human review would strengthen inference.
- **FD is not a finding of discrimination.** A nonzero absolute gap can arise for multiple reasons; a small average gap does not establish equal treatment. Examine direction, uncertainty, individual dimensions, and controls.
- **Prominence is not isolated in every pair.** Geography, role, and publisher familiarity can co-vary; distinguish clean-control comparisons from broader exploratory comparisons.
- **Factual fidelity is not the same as external truth.** A rewrite may faithfully reproduce a mistaken original source; this evaluation measures support against the supplied source, not independent fact-checking of every source claim.
- **Visibility is a separate outcome.** This benchmark cannot, on its own, establish whether an optimized publisher appears more often in generated answers.
- **Annotation and metric choices matter.** Gold-claim granularity, fabrication severity, and the FS weights affect results. Borderline claims should be audited before making strong research claims.

The experiments should be presented as an investigation of factual fidelity and possible publisher-conditioned disparity. Do **not** describe them as proving systematic publisher bias, proving the absence of bias, or showing that GEO rewriting necessarily improves fidelity.

## Repository files

Update these links to match the filenames and folders you actually commit:

- `Benchmark_Autogeomini_v2_counterbalanced.docx` — original counterbalanced 100-pair benchmark and design notes.
- `Benchmark Autogeomini with labels.docx` — benchmark version with explicit publisher and prominence labels.
- `Chatgpt 100 example run (1).xlsx` — ChatGPT evaluation/scoring workbook.
- `Claude_100_example_run_formatted.xlsx` — Claude evaluation/scoring workbook with legend, scoring, pair results, and summary.

If the repository also includes generation scripts, prompts, model identifiers, decoding parameters, gold-claim files, or analysis notebooks, document their actual filenames here. Do not claim that the repository is end-to-end reproducible unless those materials are present and runnable.

## Attribution

This is an independent research benchmark and evaluation exercise investigating GEO rewriting. The publisher-fairness scoring framework documented here is an experimental rubric used in this project; it should not be represented as an official metric published by the authors of AutoGEO.
