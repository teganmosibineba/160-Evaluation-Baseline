# 160-Evaluation-Baseline
Does LLaVA Know When It Is Wrong? A 160-Evaluation Healthcare Confidence Audit
# Does LLaVA Know When It Is Wrong?
## A 160-Evaluation Healthcare Confidence Audit

Multimodal AI systems can produce convincing answers even when their evidence is incomplete, unfamiliar, or degraded. In healthcare, a model's ability to recognize uncertainty may be as important as its ability to answer correctly.

This notebook evaluates whether LLaVA-1.5-7B's confidence corresponds to its correctness across African clinical questions and medical image question-answering tasks.

## Research Question

Can a multimodal AI model recognize when its healthcare answers may be unreliable, particularly when it encounters:

- African clinical questions;
- blurred medical images;
- darkened medical images;
- unfamiliar or difficult medical evidence?

## Why This Matters

Frontier and open-weight multimodal models are increasingly considered for healthcare, scientific research, and decision-support applications. However, many evaluation datasets are concentrated around Western clinical contexts.

When a model encounters underrepresented medical knowledge, region-specific clinical questions, or degraded visual evidence, it may produce a plausible answer without expressing appropriate uncertainty.

This project examines whether model confidence remains meaningful under these conditions.

Although the main test environment is healthcare, the evaluation methods also have relevance to biosecurity and biological research, where AI systems may need to interpret incomplete, unfamiliar, or degraded evidence safely.

## Model

- **Model:** LLaVA-1.5-7B
- **Loading method:** 4-bit NF4 quantization
- **Compute environment:** Kaggle Tesla T4 GPU
- **Inference mode:** One question at a time
- **Purpose:** Research evaluation only

## Datasets

### AfriMed-QA

AfriMed-QA provides clinical questions associated with African healthcare contexts and multiple medical specialties.

The notebook uses a custom parser that:

- reads serialized answer-option dictionaries;
- removes invalid options such as `n/a`;
- preserves the original option keys;
- prevents correct-answer indexes from shifting;
- excludes records with invalid answer mappings;
- retains available country and specialty information.

### SLAKE

SLAKE is used for medical image question answering involving radiology, anatomy, and abnormalities.

### PathVQA

PathVQA is used for pathology image question answering and fine-grained visual reasoning.

## Evaluation Design

The baseline contains:

- 20 SLAKE questions under clean, blurred, and dark conditions;
- 20 PathVQA questions under clean, blurred, and dark conditions;
- 40 AfriMed-QA clinical questions.

This produces:

- SLAKE: 20 × 3 = 60 evaluations
- PathVQA: 20 × 3 = 60 evaluations
- AfriMed-QA: 40 evaluations
- **Total: 160 evaluations**

The SLAKE and PathVQA samples are balanced between Yes and No reference answers.

## Image Conditions

Each visual question is evaluated under three conditions:

1. **Clean:** Original image
2. **Blur:** Gaussian blur applied to the image
3. **Dark:** Image brightness substantially reduced

The model is also given an explicit third option:

> Image evidence is insufficient

This permits the evaluation of whether the model abstains when visual evidence becomes less reliable.

## Confidence Method

The notebook uses option-relative confidence.

For every question:

1. The question and answer options are presented to the model.
2. Next-token logits are extracted for the available option letters.
3. A softmax is calculated across those option letters.
4. The highest-probability option becomes the model's answer.
5. That option's probability is recorded as confidence.

This confidence value is not a clinical probability. It represents the model's preference among the supplied answer choices.

## Metrics

The notebook calculates:

- accuracy;
- mean confidence;
- Expected Calibration Error;
- Brier score;
- confidence-correctness AUROC;
- abstention rate;
- high-confidence error rate;
- answer stability across image conditions;
- degradation-induced failures;
- degradation-related recoveries;
- predicted-answer distributions;
- paired confidence changes.

A high-confidence error is defined as an incorrect prediction with confidence of at least 0.80.

## Key Findings

### AfriMed-QA

Across 40 AfriMed-QA questions:

- accuracy: 27.5%;
- mean confidence: 61.6%;
- confidence-accuracy gap: 34.1 percentage points;
- incorrect answers: 29;
- high-confidence errors: 6;
- abstention rate: 0%.

A 10,000-sample bootstrap analysis estimated the 95% interval for the confidence-accuracy gap at 18.8 to 48.8 percentage points.

Because the interval remained above zero, the results support an overconfidence finding within this sample.

### PathVQA

- clean accuracy: 60%;
- blurred accuracy: 55%;
- darkened accuracy: 55%.

Blur caused three new failures and two recoveries. Darkness caused two new failures and one recovery.

Blurred PathVQA images also produced a directional answer shift:

- Yes to No changes: 5;
- No to Yes changes: 0;
- exact two-sided p-value: 0.0625.

This pattern is suggestive but does not meet the conventional 0.05 significance threshold.

### SLAKE

- clean accuracy: 75%;
- blurred accuracy: 80%;
- darkened accuracy: 70%.

Blur caused one recovery and no new failures. Darkness caused one new failure and no recovery.

The apparent improvement under blur represents only one additional correct answer and should not be interpreted as evidence that blur improves model performance.

### Abstention

The model did not select the insufficient-evidence option in any visual condition.

This suggests that the current prompting and scoring method did not produce meaningful abstention, even when images were blurred or darkened.

## Interpretation

The strongest result is substantial overconfidence on the sampled AfriMed-QA questions.

The visual results show modest prediction instability, particularly on PathVQA, but paired accuracy changes were not statistically significant.

The results do not demonstrate policy collapse. The model continued to produce varied answers and did not exhibit catastrophic performance deterioration, repetitive output, or complete answer collapse.

## Output Files

The notebook saves:

- `predictions.csv`
- `summary.csv`
- `evaluation_errors.csv`
- `audit_summary.png`
- `metadata.json`
- `incorrect_predictions.csv`
- `high_confidence_errors.csv`
- `visual_answer_distribution.csv`
- `visual_answer_stability.csv`
- `degradation_outcomes.csv`
- `paired_confidence_changes.csv`
- `paired_robustness_statistics.csv`
- `degradation_failure_details.csv`
- `degradation_recovery_details.csv`
- `afrimed_calibration_bins.csv`
- `afrimed_bootstrap_intervals.csv`

## Limitations

- AfriMed-QA is text-based rather than image-based.
- SLAKE and PathVQA are not African-origin datasets.
- The sample remains relatively small.
- Only one multimodal
