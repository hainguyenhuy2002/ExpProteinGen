# Interpretable Stability Evaluation for Generated Proteins

## 1. Motivation and Research Question

Modern protein-generation methods can create many new protein sequences. However, after a protein is generated, researchers still need to know:

* Is this protein likely to be stable?
* Which parts of the protein are sensitive to mutation?
* Which small sequence changes could improve its stability?
* Which changes are likely to damage the protein?

Therefore, this research focuses on **evaluating an already generated protein and suggesting how its sequence could be improved**.

The first version focuses specifically on **folding stability**, because large public experimental datasets are available for this property.

### Main research question

> **Given a generated protein that the model has never seen before, can we predict how different mutations will affect its folding stability and provide useful guidance for improving the protein?**

In simple terms:

$$
\text{Generated Protein}
\rightarrow
\text{Evaluate Stability}
\rightarrow
\text{Find Weak Positions}
\rightarrow
\text{Suggest Better Mutations}
$$

An additional research question can be explored later:

> If one mutation damages the protein, can we find another mutation that compensates for the damage?

This second question should only be studied if enough experimental double-mutation data are available.

---

## 2. Related Research and How This Research Builds on It

Several previous studies already provide important foundations.

### Tsuboyama et al., Nature 2023

Tsuboyama et al. created a large experimental dataset containing stability measurements for natural proteins, designed proteins, and many protein mutations.

Their work gives us the **experimental ground truth** needed to train and evaluate our system.

We can use their data to ask questions such as:

* Did this mutation increase stability?
* Did this mutation decrease stability?
* Which positions are very sensitive to mutation?
* Do two mutations interact with each other?

Therefore, our work does not need to perform new experiments at the beginning. We can first test the idea using existing public experimental measurements.

### ThermoMPNN

ThermoMPNN predicts how a mutation changes protein stability using protein structure.

This means that simply predicting mutation stability is **not new**.

Our research builds on this idea but aims to provide a more useful **protein-level evaluation**:

> Instead of predicting only one mutation at a time, evaluate many possible mutations of the generated protein and produce an interpretable map showing where the protein is stable, fragile, or potentially improvable.

### SPURS

SPURS can also predict stability changes caused by single and multiple mutations.

Therefore, predicting multiple mutations is also not enough to establish novelty.

Our research must focus more strongly on whether the model can provide **reliable editing guidance for completely unseen designed proteins**, together with explanations and uncertainty.

### How our research builds on them

The relationship can be summarized as:

$$
\text{Previous work:}
\quad
\text{Predict stability change of mutations}
$$

$$
\downarrow
$$

$$
\text{Our work:}
\quad
\text{Use these predictions to evaluate and diagnose an entire generated protein}
$$

The goal is therefore not only:

> “What is the predicted stability change of mutation A24L?”

but also:

> “For this generated protein, where are the weak positions, which mutations are worth trying, which mutations should be avoided, and how confident are we?”

---

## 3. Specific Problems This Research Solves

The research addresses four main problems.

### Problem 1 — We do not know which parts of a generated protein are fragile

A generated protein may contain positions where even a small mutation causes a large decrease in stability.

The system should identify these **sensitive positions**.

---

### Problem 2 — We do not know which mutations could improve the protein

For every position, there are many possible amino-acid substitutions.

The system should predict:

* stabilizing mutations;
* destabilizing mutations;
* mutations that probably have little effect.

This allows researchers to focus on a much smaller number of promising edits.

---

### Problem 3 — A prediction alone is difficult to understand

A model may predict:

> A24L improves stability.

But researchers also want to know what information the model used.

Therefore, the system should connect its prediction to the protein structure and show which local structural region is important for the prediction.

These explanations should be described carefully as **model explanations**, because the stability dataset does not directly provide the true physical mechanism behind every mutation.

---

### Problem 4 — The model may be unreliable on unfamiliar proteins

The most important test is not whether the system works on proteins similar to its training data.

It should be tested on **entirely unseen designed proteins**.

For example:

$$
\text{Training}
=
\text{Proteins A, B, C, D}
$$

$$
\text{Testing}
=
\text{Protein X}
$$

No mutations from Protein X should appear during training.

This allows us to test whether the evaluator can actually be used on a newly generated protein.

---

## 4. Specific Input and Output

### Input

The main input is:

1. **Protein sequence**

   * The amino-acid sequence of the generated protein.

2. **Protein structure**

   * An experimentally determined structure or a predicted structure such as an AlphaFold structure.

Optional input can include positions that researchers do not want to modify.

The researcher does **not** need to provide experimental stability measurements when using the system.

Experimental measurements are only required for training and evaluating the model.

### Output

The main output is an **editing map of the generated protein**.

For example:

| Position | Mutation | Prediction             | Confidence |
| -------- | -------- | ---------------------- | ---------- |
| 12       | V → L    | Stabilizing            | High       |
| 12       | V → D    | Destabilizing          | High       |
| 24       | A → V    | Slightly stabilizing   | Medium     |
| 37       | F → A    | Strongly destabilizing | High       |

The complete result can be represented as a heatmap:

$$
\text{Protein positions}
\times
\text{Possible amino-acid substitutions}
$$

Each cell represents the predicted effect of one mutation.

The system should return four main outputs:

**1. Editing map**

Shows the predicted effect of possible mutations across the protein.

**2. Sensitive positions**

Identifies regions where mutations are likely to strongly damage stability.

**3. Recommended edits**

Identifies mutations that may improve folding stability.

**4. Structural explanation and confidence**

Shows which structural region is related to a prediction and how confident the system is.

Therefore, the overall pipeline is:

$$
\boxed{\text{Generated Protein}}
$$

$$
\downarrow
$$

$$
\boxed{\text{Sequence + Structure}}
$$

$$
\downarrow
$$

$$
\boxed{\text{Stability Evaluator}}
$$

$$
\downarrow
$$

$$
\boxed{
\text{Editing Map}
+
\text{Weak Positions}
+
\text{Suggested Mutations}
+
\text{Confidence}
}
$$

These outputs follow the original proposal's idea of an editing map, structural explanation, edit comparison, and evidence/limitations.

---

## 5. Public Datasets

We will mainly use **two public experimental protein-stability datasets**.

### Dataset 1 — Tsuboyama et al., Nature 2023

**Mega-scale experimental analysis of protein folding stability in biology and design**

This will be the main dataset for learning and evaluating **mutation effects**.

The dataset contains experimental folding-stability measurements for both natural and designed proteins. Importantly, it provides large mutation landscapes: the published dataset includes comprehensive single mutations for hundreds of protein domains and double mutations for selected residue pairs.

For our research, it provides:

* protein sequences;
* experimentally measured folding stability;
* single-amino-acid mutations;
* double mutations;
* designed proteins;
* predicted protein structures.

This allows us to study:

$$
\text{Original Protein}
+
\text{Mutation}
\rightarrow
\text{Change in Stability}
$$

and therefore train and test the **editing map** produced by our system.

Useful files include:

* `Tsuboyama2023_Dataset2_Dataset3_20230416.csv`
* `Single_DMS_list.csv`
* `Double_DMS_list.csv`
* `AlphaFold_model_PDBs.zip`

**Public access:**
[Tsuboyama 2023 MegaScale dataset — Zenodo](https://zenodo.org/records/7992926?utm_source=chatgpt.com)

The Zenodo release contains the processed stability datasets, structures, and analysis files used in the Nature study.

---

### Dataset 2 — Rocklin et al., Science 2017 / TAPE Stability Dataset

**Global analysis of protein folding using massively parallel design, synthesis, and testing**

This dataset is especially relevant because it focuses directly on **de novo designed proteins**.

Rocklin et al. experimentally measured folding and stability for:

* more than **15,000 de novo designed miniproteins**;
* about **10,000 point mutants**;
* natural-protein controls;
* negative-control sequences.

More than 2,500 experimentally stable designed proteins were identified.

This makes the dataset useful for our main application:

$$
\boxed{\text{Newly Generated Protein}}
\rightarrow
\boxed{\text{Evaluate Its Stability}}
$$

It can also help us test whether the model understands how small sequence changes around a designed protein affect stability.

A processed version of this dataset is used as the **Stability task in TAPE**. TAPE provides protein sequences together with experimental stability scores and predefined train, validation, and test splits.

**Public access:**

[Original TAPE repository and Stability dataset](https://github.com/songlab-cal/tape-neurips2019?utm_source=chatgpt.com)

A convenient downloadable processed version is also available here:

[Processed Rocklin/TAPE Stability dataset — Hugging Face](https://huggingface.co/datasets/proteinglm/stability_prediction?utm_source=chatgpt.com)

The processed version contains approximately **53,614 training, 2,512 validation, and 12,851 test sequences**, each associated with an experimental stability score.

---

### How the Two Datasets Will Be Used

The two datasets provide complementary information.

**Tsuboyama 2023** is most useful for:

$$
\boxed{\text{Which mutation improves or damages this protein?}}
$$

because it contains large single- and double-mutation landscapes.

**Rocklin 2017** is most useful for:

$$
\boxed{\text{Can the system evaluate newly designed proteins?}}
$$

because it contains thousands of experimentally tested de novo protein designs.

Therefore, together they support the complete research goal:

$$
\text{Generated Protein}
\rightarrow
\text{Stability Evaluation}
\rightarrow
\text{Mutation Map}
\rightarrow
\text{Suggested Improvements}
$$

Before treating Rocklin as a fully independent test dataset, sequence overlap with the selected Tsuboyama training set should be checked and removed.


# Research Direction in One Sentence

> **Develop a system that takes a newly generated protein sequence and structure, evaluates how stable different parts of the protein are, predicts which small mutations could improve or damage its folding stability, and presents the results as an interpretable editing map that can be tested against public experimental data.**
