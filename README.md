# Medical Specialty Classification — NLP Progression Pipeline

This repository documents the step-by-step iterative development of an NLP classification system designed to predict medical specialties from clinical text and transcription data. 

To keep our methodology transparent, reproducible, and easy to reference in the final report, each iteration is organized in its own self-contained notebook demonstrating the specific hypothesis, implementation, and performance gains.

---

## Iteration Overview & Results

| # | Notebook | Core Strategy | Accuracy | Macro F1 | Primary Contribution |
|---|---|---|:---:|:---:|---|
| **00** | `00_baseline_naive_bayes.ipynb` | Standard TF-IDF + MultinomialNB | **28.69%** | **0.1601** | Initial benchmark on raw transcription text. |
| **01** | `01_linearsvc_multifield.ipynb` | Multi-field text concatenation + LinearSVC | **41.37%** | **0.3013** | Combined transcription, description, and keywords. |
| **02** | `02_weighted_features_linearsvc.ipynb` | Targeted keyword & name weighting | **44.31%** | **0.3541** | Doubled token weights for high-signal clinical terms. |
| **03** | `03_mlm_data_augmentation.ipynb` | Bio_ClinicalBERT Masked LM generation | **45.78%** | **0.3705** | Contextual synonym replacement for minority classes. |
| **04** | `04_calibrated_decision_margins.ipynb` | Class weighting & decision boundary tuning | **47.94%** | **0.4118** | Shifted decision margins to recover tail classes. |
| **05** | `05_finetuned_clinicalbert.ipynb` | Full fine-tuning of `Bio_ClinicalBERT` | **92.16%** | **0.8920** | Domain-specific transformer with cost-sensitive loss. |

---

## Notebook Descriptions & Rationale

### 1. `00_baseline_naive_bayes.ipynb` (Baseline Benchmark)
* **Goal:** Establish a baseline classification benchmark.
* **Method:** Cleaned and stemmed medical transcriptions using unigram/bigram TF-IDF vectorization with a `MultinomialNB` classifier.
* **Findings:** Naive Bayes independence assumptions struggled heavily with long clinical narratives and class imbalance (Macro F1: ~0.16).

### 2. `01_linearsvc_multifield.ipynb` (Multi-Field Feature Expansion)
* **Goal:** Leverage all available textual metadata rather than relying solely on transcriptions.
* **Method:** Concatenated `transcription`, `description`, and `keywords` into a single representation and switched the classifier to `LinearSVC` with balanced class weights.
* **Findings:** Resulted in an immediate +12.68% boost in accuracy and doubled Macro F1, showing that concise physician summaries contain high diagnostic signal.

### 3. `02_weighted_features_linearsvc.ipynb` (Feature Weighting)
* **Goal:** Emphasize high-density clinical keywords over long, diffuse transcription narratives.
* **Method:** Engineered a weighted input sequence: `2 × sample_name + 2 × keywords + description + transcription`.
* **Findings:** Raised accuracy to 44.31% by preventing critical specialty terminology from being diluted across lengthy records.

### 4. `03_mlm_data_augmentation.ipynb` (Contextual Augmentation)
* **Goal:** Mitigate severe class imbalance for rare medical specialties without overfitting.
* **Method:** Used `emilyalsentzer/Bio_ClinicalBERT`'s Masked Language Model (MLM) to dynamically mask and replace tokens with context-appropriate medical synonyms for underrepresented classes.
* **Findings:** Generated clinically sound synthetic samples and lifted Macro F1 to 0.3705.

### 5. `04_calibrated_decision_margins.ipynb` (Decision Boundary Calibration)
* **Goal:** Correct for SVM hyperplane bias favoring majority classes.
* **Method:** Computed class-frequency cost matrices ($C_k$) and adjusted decision thresholds to ensure minority categories met the evaluation cutoff.
* **Findings:** Achieved 47.94% accuracy with balanced per-class recall across minority classes.

### 6. `05_finetuned_clinicalbert.ipynb` (Final Model & Submission Generation)
* **Goal:** Transition from static n-grams to deep contextual representation via end-to-end transformer fine-tuning.
* **Method:** Fine-tuned `Bio_ClinicalBERT` using a Head+Tail token truncation strategy (capturing both introductory clinical reasons and concluding diagnoses), cost-sensitive Cross-Entropy loss, and scheduled learning rate warmup.
* **Findings:** Achieved a final validation accuracy of **92.16%** and generated the final test predictions (`results.txt`).

---

## Repository Structure

```text
├── data/
│   ├── train.csv                              # Training dataset
│   └── test.csv                               # Unlabeled test dataset
├── notebooks/
│   ├── 00_baseline_naive_bayes.ipynb          # MultinomialNB baseline
│   ├── 01_linearsvc_multifield.ipynb          # Multi-field concatenation + LinearSVC
│   ├── 02_weighted_features_linearsvc.ipynb   # Weight-engineered features
│   ├── 03_mlm_data_augmentation.ipynb         # Bio_ClinicalBERT MLM augmentation
│   ├── 04_calibrated_decision_margins.ipynb   # Margin calibration & cost tuning
│   └── 05_finetuned_clinicalbert.ipynb        # Final model & test set inference
├── results.txt                                # Final predictions for submission
└── README.md