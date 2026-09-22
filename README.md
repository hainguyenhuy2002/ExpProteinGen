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

## 5. Public Dataset

### Main Dataset: Tsuboyama et al., Nature 2023

The main dataset for this project is:

**Mega-scale experimental analysis of protein folding stability in biology and design**

This dataset is particularly suitable because it contains:

* natural proteins;
* computationally designed proteins;
* experimental folding-stability measurements;
* large numbers of single mutations;
* double mutations;
* predicted protein structures.

Important public files include:

* `Tsuboyama2023_Dataset2_Dataset3_20230416.csv`
* `Single_DMS_list.csv`
* `Double_DMS_list.csv`
* `AlphaFold_model_PDBs.zip`

The single-mutation data can be used for the main project:

$$
\text{Sequence + Structure + Mutation}
\rightarrow
\text{Experimental Stability Change}
$$

This allows us to train the model and compare its predicted editing map with the experimentally measured mutation landscape.

The double-mutation data can later be used to study **mutation interactions and compensating mutations**.

### ProteinGym

ProteinGym can be used as an additional mutation-effect benchmark when the assay measures a relevant property.

However, activity, binding, abundance, and folding stability are different properties. Therefore, only suitable stability-related assays should be used.

Overlap with the Tsuboyama dataset must also be removed before treating ProteinGym as an independent test dataset.

---

# Research Direction in One Sentence

> **Develop a system that takes a newly generated protein sequence and structure, evaluates how stable different parts of the protein are, predicts which small mutations could improve or damage its folding stability, and presents the results as an interpretable editing map that can be tested against public experimental data.**
