# Interpretable Stability Evaluation for Generated Proteins

## 1. Motivation and Research Question

Modern protein-generation models can create many new protein sequences. However, after a protein is generated, researchers still need to know:

* Is this protein likely to be stable?
* Which parts of the protein are sensitive or fragile?
* Which mutations could improve its stability?
* Which mutations should be avoided?
* **Why does the model think a mutation is good or bad?**

Most existing methods mainly provide a prediction score. A researcher may know that a mutation is predicted to improve stability, but may not understand **what part of the sequence or structure caused the prediction**.

Therefore, this research focuses on building an **interpretable evaluator for generated proteins**.

The first version focuses specifically on **folding stability**, because large public experimental datasets are available for this property.

### Main Research Question

> **Given a newly generated protein, can we predict which mutations will improve or damage its folding stability, while also explaining which parts of the protein structure are most important for each prediction?**

In simple form:

$$
\text{Generated Protein}
\rightarrow
\text{Stability Evaluation}
\rightarrow
\text{Mutation Suggestions}
\rightarrow
\boxed{\text{Interpretation}}
$$

The system should not only answer:

> “Mutation A24L is predicted to improve stability.”

It should also provide:

> “The prediction mainly depends on the local structural environment around residue 24 and its nearby residues.”

---

# 2. Related Research and How This Research Builds on It

## Tsuboyama et al., Nature 2023

Tsuboyama et al. created a large experimental dataset containing:

* natural proteins;
* designed proteins;
* single mutations;
* double mutations;
* experimentally measured folding stability;
* predicted protein structures.

This work provides the experimental data needed to learn:

$$
\text{Mutation}
\rightarrow
\text{Change in Stability}
$$

It allows us to test whether our model correctly predicts which mutations stabilize or destabilize a designed protein.

Their work gives us the experimental ** ground truth** needed to train and evaluate our system.


---

## Rocklin et al., Science 2017

Rocklin et al. experimentally tested thousands of **de novo designed proteins** and many sequence variants.

This dataset is especially useful because our research focuses on:

$$
\boxed{\text{Evaluating generated/designed proteins}}
$$

rather than only natural proteins.

Rocklin's data allow us to test whether the model can evaluate proteins that were intentionally designed from scratch.

---

## ThermoMPNN

ThermoMPNN predicts how mutations change protein stability using protein structure.

Therefore:

> **Predicting stability change alone is not enough to make our research new.**

Our work builds on this by turning mutation prediction into a more complete evaluation of a generated protein.

Instead of only returning:

$$
\Delta\Delta G = -1.2
$$

our system aims to return:

* which positions are sensitive;
* which mutations are promising;
* which mutations are dangerous;
* how confident the prediction is;
* **which structural regions influenced the prediction.**

---

## SPURS

SPURS also predicts stability changes caused by single and multiple mutations.

This means that simply adding multiple mutations is also not sufficient as a research contribution.

Our research therefore focuses more strongly on:

$$
\boxed{\text{Prediction + Protein-Level Evaluation + Interpretability}}
$$

rather than prediction alone.

---

# 3. Specific Problems This Research Solves

## Problem 1 — We do not know which parts of a generated protein are fragile

A newly generated protein may contain positions where small sequence changes strongly reduce stability.

The system should identify:

$$
\boxed{\text{Sensitive positions}}
$$

For example:

| Position | Sensitivity |
| -------- | ----------- |
| 12       | High        |
| 24       | Low         |
| 37       | High        |
| 42       | Medium      |

This helps researchers understand which parts of the generated protein are robust and which parts require caution.

---

## Problem 2 — We do not know which mutations could improve the protein

For each position, many amino-acid substitutions are possible.

The system should predict whether each mutation is:

* stabilizing;
* neutral;
* destabilizing.

For example:

| Mutation | Predicted Effect       |
| -------- | ---------------------- |
| A24L     | Stabilizing            |
| A24V     | Slightly stabilizing   |
| A24D     | Strongly destabilizing |
| F37A     | Destabilizing          |

This helps researchers select a small number of useful mutations instead of experimentally testing every possibility.

---

## Problem 3 — Existing Predictions Are Difficult to Interpret

This is one of the main focuses of the proposed research.

A prediction such as:

> “A24L improves stability.”

is useful, but it does not tell the researcher **why the model made this prediction**.

Our system should therefore provide an **interpretation for each important prediction**.

### Example

Instead of only returning:

> A24L → predicted stabilizing

the system could return:

> A24L → predicted stabilizing
> Important region: residues 21–28
> Important neighboring residues: V21, F27, L28
> Model confidence: high

The corresponding 3D structure can highlight these residues.

Therefore, the researcher can see:

$$
\text{Mutation}
\rightarrow
\text{Prediction}
\rightarrow
\boxed{\text{Important structural region}}
$$

### Interpretation Output

The interpretation component could include:

* important residues around the mutation;
* important residue–residue relationships;
* local structural environment;
* whether the residue is buried or exposed;
* which structural region contributes most strongly to the prediction;
* model confidence.

For example:

$$
\boxed{\text{A24L}}
$$

$$
\downarrow
$$

**Predicted effect:** Stabilizing

**Important region:** Local environment around residue 24

**Important residues:** 21, 27, 28

**Confidence:** High

### Important Limitation

The interpretation should be described as:

> **what information the model used to make its prediction**

rather than automatically claiming:

> **the true physical mechanism causing the stability change.**

For example, the model may identify residues 24 and 27 as important.

But the available mutation dataset does not necessarily prove:

> “Residue 24 forms a new hydrogen bond with residue 27 and this causes the stability increase.”

The experimental dataset mainly tells us whether stability changed, not the exact physical reason why.

Therefore, our research focuses on **model interpretability that can be checked against experimental mutation patterns**, rather than claiming full physical explanation.

---

## Problem 4 — The Evaluator Must Work on Unseen Generated Proteins

The model should not only work on proteins that appeared during training.

The real application is:

$$
\boxed{\text{New generated protein}}
$$

that the model has never seen before.

Therefore, training and testing should be separated by protein.

For example:

### Training

```text
Protein A
Protein B
Protein C
Protein D
```

### Testing

```text
Protein X
```

No mutations from Protein X should appear in the training dataset.

Then we can test:

> Can the system correctly evaluate the mutation landscape of a completely unseen designed protein?

This follows the proposal's requirement to evaluate on held-out protein backgrounds.

---

# 4. Specific Input and Output

## Input

The system takes:

### 1. Protein Sequence

The amino-acid sequence of the newly generated protein.

Example:

```text
MKTAYIAKQRQISFVKSHFSRQ...
```

### 2. Protein Structure

A 3D structure of the protein.

This can be:

* experimentally determined structure; or
* predicted structure such as an AlphaFold prediction.

Optional input can include positions that researchers do not want to modify.

The researcher does **not** need experimental stability measurements when using the trained system.

---

## Output

The system produces four major outputs.

### Output 1 — Stability Editing Map

The system evaluates many possible mutations across the protein.

The result can be represented as:

$$
\text{Protein positions}
\times
\text{Possible amino-acid substitutions}
$$

Each cell represents the predicted effect of one mutation.

Example:

| Position |       A |       V |       L |  D |  F |
| -------- | ------: | ------: | ------: | -: | -: |
| 12       | Neutral |       + |      ++ | -- |  - |
| 13       |       - | Neutral |       + | -- |  + |
| 14       |      ++ |       + | Neutral |  - | -- |

This creates an **editing map for the entire generated protein**.

---

### Output 2 — Sensitive and Important Positions

The model identifies positions where mutations strongly affect stability.

For example:

> Position 37 is highly sensitive. Most substitutions are predicted to destabilize the protein.

This helps researchers identify fragile regions.

---

### Output 3 — Suggested Mutations

The model provides promising edits.

Example:

| Suggested Mutation | Predicted Stability Effect | Confidence |
| ------------------ | -------------------------: | ---------- |
| A24L               |                       +1.2 | High       |
| D37N               |                       +0.8 | Medium     |
| V42I               |                       +0.5 | High       |

These mutations can then be tested experimentally.

---

### Output 4 — Interpretation of the Prediction

This is a central output of the project.

For every important suggested or risky mutation, the system should show **what parts of the protein contributed to the prediction**.

For example:

#### Mutation

$$
A24L
$$

#### Prediction

$$
\text{Stabilizing}
$$

#### Interpretation

```text
Important position: residue 24

Important surrounding residues:
21, 23, 27, 28

Important structural region:
local packed region around residue 24

Model confidence:
High
```

The same information can be shown on the protein's 3D structure by highlighting the relevant residues.

Therefore, the final output is:

$$
\boxed{
\text{Editing Map}
+
\text{Sensitive Positions}
+
\text{Suggested Mutations}
+
\textbf{Structural Interpretation}
+
\text{Confidence}
}
$$

This extends the original proposal's editing-map and structural-explanation outputs.

---

# 5. Public Datasets

### Main Dataset 1 — Tsuboyama et al., Nature 2023

**Mega-scale experimental analysis of protein folding stability in biology and design**

This is the main dataset for learning and evaluating mutation effects.

It contains:

* natural proteins;
* computationally designed proteins;
* experimental folding-stability measurements;
* large numbers of single mutations;
* double mutations;
* predicted structures.

It can support:

$$
\text{Original Protein}
+
\text{Mutation}
\rightarrow
\text{Experimental Stability Change}
$$

This allows us to compare the model's predicted editing map with experimentally measured mutation effects.

Important public files include:

* `Tsuboyama2023_Dataset2_Dataset3_20230416.csv`
* `Single_DMS_list.csv`
* `Double_DMS_list.csv`
* `AlphaFold_model_PDBs.zip`

**Public access:**
https://zenodo.org/records/7992926

---

### Main Dataset 2 — Rocklin et al., Science 2017 / TAPE Stability

**Global analysis of protein folding using massively parallel design, synthesis, and testing**

This dataset is especially relevant because it contains a large number of **de novo designed proteins**.

It includes:

* more than 15,000 designed miniproteins;
* thousands of experimentally stable designs;
* approximately 10,000 point-mutant variants;
* experimentally measured stability information.

This dataset helps answer:

$$
\boxed{
\text{Can the evaluator work on designed/generated proteins?}
}
$$

It can therefore complement Tsuboyama:

### Tsuboyama

Best suited for:

$$
\text{Mutation}
\rightarrow
\text{Stability Change}
$$

### Rocklin

Best suited for:

$$
\text{Designed Protein}
\rightarrow
\text{Experimental Stability Evaluation}
$$

A processed version is available through the TAPE Stability benchmark.

**Public access:**

Original TAPE dataset:

https://github.com/songlab-cal/tape-neurips2019

Processed Stability dataset:

https://huggingface.co/datasets/proteinglm/stability_prediction

Before using Rocklin as an independent test set, sequence overlap with proteins used in the Tsuboyama training set should be removed.

---

# Final Research Pipeline

The complete research can therefore be summarized as:

$$
\boxed{\text{Newly Generated Protein}}
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
\boxed{\text{Interpretable Stability Evaluator}}
$$

$$
\downarrow
$$

$$
\begin{array}{c}
\text{Stability Editing Map}\\
+\\
\text{Sensitive Positions}\\
+\\
\text{Suggested Mutations}\\
+\\
\mathbf{\text{Structural Interpretation}}\\
+\\
\text{Prediction Confidence}
\end{array}
$$

The main goal is therefore not only:

> **“Can we predict whether a mutation changes protein stability?”**

but:

> **“Can we evaluate a newly generated protein, identify useful and risky mutations, and explain which parts of its structure influence those predictions?”**
