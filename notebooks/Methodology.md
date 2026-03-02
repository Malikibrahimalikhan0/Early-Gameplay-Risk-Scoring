# Methodology Documentation

## 1. Problem Framing

The objective of this project is to develop an early-session risk scoring system that estimates the probability of incorrect outcomes using early gameplay telemetry data.

The core hypothesis is that behavioral signals within the first 20 actions of a session contain predictive information about future correctness.


## 2. Data Preparation

The dataset consists of:

- Question-level labels (correct / incorrect)
- Session-level gameplay logs

Because multiple questions belong to the same gameplay session, session identifiers were extracted to construct a grouping variable (`base_session_id`).

This grouping variable was later used to prevent data leakage during model evaluation.

Due to computational constraints, gameplay logs were processed in chunks and a fraction of records was sampled for feature engineering.


## 3. Feature Engineering

Two categories of features were created:

### 3.1 Overall Session Features

Aggregated at the session + level group level:

- Number of events
- Total session time
- Mean and median time between actions
- Number of long pauses (> 5000 ms)
- Unique event count
- Pause ratio (long pauses / total events)

These features capture engagement intensity and behavioral pacing.

### 3.2 Early-Session Features (First 20 Actions)

Only the first 20 actions of each session were used to simulate early detection.

Engineered features include:

- Early mean time between actions
- Early median time
- Early timing volatility (standard deviation)
- Early long pauses
- Early unique events
- Early pause ratio

These features aim to capture early hesitation or instability.


## 4. Train-Test Splitting Strategy

A group-based train-test split was applied using `GroupShuffleSplit` on `base_session_id`.

This prevents multiple questions from the same gameplay session appearing in both training and testing sets.

Initial experiments using naive random splitting resulted in inflated performance, confirming the presence of potential leakage.

Group-based validation provides a more realistic estimate of model performance.


## 5. Modeling Approach

Two models were evaluated:

### Logistic Regression
- Baseline linear classifier
- Class imbalance handled using `class_weight="balanced"`

### Random Forest
- 200 trees
- Maximum depth of 10
- Parallel processing enabled
- Class imbalance handled using `class_weight="balanced"`

Performance was evaluated using ROC AUC.


## 6. Risk Scoring

The Random Forest model was used to generate a probability score for incorrect outcomes.

Risk buckets were created using quantile-based binning into five categories:

- Very Low
- Low
- Medium
- High
- Very High

Correctness rates were analyzed across buckets to evaluate ranking effectiveness.


## 7. Intervention Simulation

Two strategies were evaluated:

1. Top 20% highest-risk sessions
2. Threshold optimization to balance recall and intervention size

Results show that targeted intervention captures a substantial portion of incorrect outcomes while limiting the number of sessions flagged.


## 8. Limitations

- Only 20% of gameplay logs were sampled
- Dataset is imbalanced (~14% incorrect)
- Later level groups have limited data
- Model performance remains moderate (AUC ≈ 0.64)

This system should be viewed as a research prototype rather than a production-ready solution.
