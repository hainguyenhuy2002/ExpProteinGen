# Interpretable stability editing for designed proteins

**Working direction:** Help researchers understand where a designed protein is sensitive to changes and identify small sequence edits that improve its folding stability.

**Recommendation:** Pursue a public-data feasibility study of this direction. The prediction task is data-feasible; a distinctive research contribution still needs to be demonstrated against strong existing approaches.

Prepared 17 September 2026. This document defines the problem, inputs, outputs and evidence requirements; it does not prescribe a model architecture.

## 1. The purpose in simple language

A protein is a chain of building blocks called amino acids. For many applications, that chain needs to maintain a folded shape. Here, **folding stability** means how strongly the folded state is favored over the unfolded state under specified conditions. It is one property relevant to usefulness, not a guarantee of biological function.

Imagine a researcher already has a designed protein. The practical question is:

> **Which small changes could make this protein more stable, which changes should we avoid, and what evidence supports those conclusions?**

The proposed output is an experimentally testable **map of the consequences of editing one protein**. Its value would be helping researchers plan modifications and avoid damaging changes. A numerical prediction is useful within that map, but a ranking of unrelated proteins is not the central deliverable.

Protein engineering already uses stability prediction and design. The research opportunity must therefore be a demonstrable improvement in the reliability and explanatory value of editing guidance. [ThermoMPNN, PNAS 2024](https://doi.org/10.1073/pnas.2314853121)

## 2. How this relates to our earlier discussion

We keep the broad objective: **evaluate designed proteins and provide useful interpretation**.

We make one explicit scope change: the first project concerns **folding stability**, rather than whether a binder attaches to a target or whether an enzyme performs a reaction. This is a data-driven choice: public stability experiments include many measured sequence changes, which can test the proposed editing guidance.

The initial population is **small designed proteins and protein domains within the available data's scope**. “Designed” includes computationally designed proteins from older studies; it does not mean that every example comes from a recent AI generator.

This project would not initially claim to diagnose every cause of failure, preserve binding or enzyme activity after an edit, or work on outputs from every generator.

## 3. The research question

> **For a designed protein unlike those used in training, can we provide an accurate, interpretable map of stabilizing and destabilizing sequence changes that helps researchers choose useful edits?**

The more ambitious extension is:

> **When one edit destabilizes a protein, can we identify a different, compensating edit that restores some of the lost stability—and predict when that compensation will fail?**

The extension is particularly compelling because it asks for a testable correction. It is conditional on having enough matched experimental examples. Predicting compensation is also not automatically novel.

## 4. Input: what the researcher supplies

| Input | Meaning | Required in the first version? |
|---|---|---|
| One protein sequence | The amino-acid letters describing the existing candidate | Yes |
| Its 3D structure or a predicted structure | Where the building blocks are positioned in space | Yes for the proposed structure-based explanation output; predicted structures must be identified as predictions |
| Editing constraints | Positions that must remain unchanged, or edits the researcher wants to examine | Optional |
| Intended measurement conditions | The experimental setting for which the prediction is meaningful | Fixed to the benchmark setting initially; record metadata where available |

For the compensation extension, the input additionally identifies the destabilizing edit or the already edited sequence.

**The user would not need to supply experimental stability labels at prediction time.** Those labels are needed to develop and evaluate the system.

You can use proteins released by other papers. There is no requirement to build a generator or generate new sequences for the initial study.

## 5. Output: what the system returns

| Output | Presentation | What it enables |
|---|---|---|
| **Editing map** | A table or heatmap: sequence positions on one axis, possible replacement amino acids on the other | See which changes are predicted to improve stability, reduce it, or have little effect |
| **Structural explanation** | A small highlighted region or set of relationships in the 3D structure, linked to a particular edit | Inspect what the model uses to support its prediction |
| **Edit comparison** | Original sequence versus proposed edited sequence, predicted stability change, and uncertainty | Compare concrete modifications to the same candidate |
| **Evidence and limits** | State whether an explanation is model-derived, experimentally corroborated, or insufficiently supported | Avoid confusing plausible explanations with established mechanisms |

**Illustrative output, not a result:**

> “Replacing the amino acid at position 24 is predicted to improve stability. The prediction depends on its local structural environment. Evidence is weaker here because this environment differs from our validated examples.”

For the extension, an output could instead say that an edit at position 24 is predicted to compensate for damage caused by an edit at position 47. That statement must be tested using experiments containing both edits.

We should not promise a detailed physical explanation such as “this hydrogen bond causes the improvement” from mutation measurements alone. Those measurements establish the consequences of sequence changes; they do not uniquely identify the physical mechanism.

## 6. Public data to start with

### Primary resource: Tsuboyama et al., Nature 2023

**Mega-scale experimental analysis of protein folding stability in biology and design** provides a large experimental foundation involving natural and designed proteins and many sequence variants. The released files include processed stability estimates, single- and double-mutation lists, quality-filtering scripts and predicted structures. Stability is inferred through a proteolysis assay and its fitted model; it is not a direct recording of protein folding. [Paper](https://www.nature.com/articles/s41586-023-06328-6) · [Public data](https://zenodo.org/records/7992926)

Start with these released files:

- `Processed_K50_dG_datasets.zip`
- `Tsuboyama2023_Dataset2_Dataset3_20230416.csv` within that archive
- `Single_DMS_list.csv` and `Double_DMS_list.csv`
- `AlphaFold_model_PDBs.zip`

The study is large at the **variant-measurement level**. That must not be described as hundreds of thousands of unrelated designed proteins. ThermoMPNN's published analysis explicitly includes both natural and de novo proteins, confirming that designed backgrounds are represented. [ThermoMPNN paper and figures](https://pubmed.ncbi.nlm.nih.gov/38285937/)

### The exact records we need

| Research claim | Necessary experimental records |
|---|---|
| An edit improves stability | Reference sequence and edited sequence with comparable stability measurements |
| A position is sensitive to changes | Several measured replacements at that position |
| Two edits interact | Reference, edit A alone, edit B alone, and A+B, measured comparably |
| Edit B compensates for edit A | The same matched set, showing that A+B improves on A; full restoration requires an additional comparison with the reference |
| Guidance transfers to unfamiliar designs | Entire held-out protein backgrounds, separated from close training relatives |
| Guidance transfers across generators | Reliable generator provenance and independent experimental examples from multiple generators |

Each analysis also needs parent-protein identifiers, natural/designed annotations, sequence-to-structure mappings, measurement quality flags and uncertainty where supplied. Keep the source's sign convention explicit when reporting stability changes; different datasets use different conventions.

**Still to audit:** the exact number of usable designed backgrounds, beneficial single edits and complete four-member experimental sets after filtering. The public release supports starting the study, but these subset sizes have not been verified here.

### Resources that should not be the main foundation

ProteinGym supplies public mutation-effect benchmarks and existing predictions. Only assays matching the chosen property are appropriate, and overlap with Tsuboyama-derived data must be removed before calling any test independent. Binding, abundance and activity labels are not interchangeable with folding-stability measurements. [ProteinGym](https://github.com/OATML-Markslab/ProteinGym)

Unlabelled outputs from recent protein generators can demonstrate input compatibility. They cannot demonstrate that an explanation or suggested improvement is experimentally correct.

## 7. What is feasible, and what remains conditional?

| Claim | Assessment |
|---|---|
| Build and retrospectively evaluate single-edit stability maps using public data | **Feasible** |
| Include computationally designed proteins | **Feasible**, with final subset size to be audited |
| Evaluate on unfamiliar protein backgrounds | **Feasible in principle**, with careful separation of related proteins |
| Validate selected compensating edits | **Conditional** on enough complete matched measurements |
| Explain the exact physical mechanism of every improvement | **Unsupported by these labels alone** |
| Prove broad transfer across modern generators | **Not established by this starting dataset** |
| Guarantee improved binding, enzyme activity or therapeutic usefulness | **Outside the first project's scope** |

All initial experimental evaluation can be retrospective: hide published measurements, make predictions, and then compare with the hidden results. Recommendations outside the measured variant collection remain untested proposals.

## 8. How this builds on existing research

| Existing study | What it already supplies | Consequence for our proposal |
|---|---|---|
| **Tsuboyama et al., Nature 2023** | Experimental stability landscapes and analysis of mutation effects | Reuse these observations as evidence; visualizing a landscape alone is insufficient novelty |
| **ThermoMPNN, PNAS 2024** | Structure-based prediction of stability changes and support for stability design | Predicting helpful single edits is an existing capability and a necessary baseline |
| **SPURS, Nature Communications 2026; online December 2025** | Single- and multiple-mutation stability prediction, identification of stabilizing changes, and functional-site analyses | Neither adding multiple edits nor highlighting sites establishes novelty by itself |

Sources: [Tsuboyama](https://www.nature.com/articles/s41586-023-06328-6), [ThermoMPNN](https://doi.org/10.1073/pnas.2314853121), [SPURS](https://www.nature.com/articles/s41467-025-67609-4).

The **candidate contribution to investigate** is whether concise, experimentally testable explanations support reliable editing decisions on unfamiliar designed proteins. This literature check establishes close overlap; it does not establish an unoccupied research gap.

## 9. What would make the result compelling?

A strong result would show that the system's editing guidance improves stability more reliably than strong existing predictors on held-out designed backgrounds, and that its explanations predict measured consequences of changes rather than merely looking plausible.

For the compensation extension, the clearest demonstration would be:

> “On unseen protein backgrounds, the system identifies edits that compensate for destabilizing changes, predicts when the same edit stops helping, and gives a compact explanation consistent with independent mutation measurements.”

This is a proposed success criterion, not a result we already have. A physical contact explanation would require additional evidence beyond those mutation outcomes.

The project should separately assess prediction accuracy, faithfulness of the explanation to the model, and experimental support for the editing advice. Improvement in one does not automatically prove the others.

## 10. The first decision before developing a model

Audit the primary dataset and establish a baseline on held-out designed proteins. Count independent backgrounds, useful edits and matched mutation combinations, then check whether existing predictors already provide the intended editing guidance.

Proceed toward a full research proposal if there is enough independent experimental coverage and a measurable weakness that the proposed explanation output could help address. If existing predictors already solve the selected task, a new interface or heatmap would not justify the claimed contribution.

**Final working statement:**

> Develop an interpretable evaluator for editing the folding stability of designed proteins. Given a sequence and structure, it predicts the consequences of small changes, identifies useful and risky edits, and makes its explanations testable against public experiments. Start with single edits; investigate compensating edits only where matched data support them.

This offers a feasible public-data starting point and a concrete improvement-oriented objective. Its scientific strength will depend on validated editing utility and differentiation from existing predictors, not on presenting interpretability as novel by itself.
