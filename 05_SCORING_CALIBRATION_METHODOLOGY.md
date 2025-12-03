# SCORING CALIBRATION METHODOLOGY

## Meridian Sentinel - How to Derive, Validate & Calibrate Fraud Detection Weights

**Document Version:** 1.0  
**Date:** December 2024  
**Classification:** Technical Calibration Guide

---

## TABLE OF CONTENTS

- [SCORING CALIBRATION METHODOLOGY](#scoring-calibration-methodology)
  - [Meridian Sentinel - How to Derive, Validate \& Calibrate Fraud Detection Weights](#meridian-sentinel---how-to-derive-validate--calibrate-fraud-detection-weights)
  - [TABLE OF CONTENTS](#table-of-contents)
  - [COMPLETE SCORING SUMMARY (Official Fraud Detection Logic)](#complete-scoring-summary-official-fraud-detection-logic)
    - [All 7 Indicators at a Glance](#all-7-indicators-at-a-glance)
    - [Performance Targets](#performance-targets)
  - [1. INTRODUCTION: UNDERSTANDING THE SCORING PROBLEM](#1-introduction-understanding-the-scoring-problem)
    - [1.1 The Core Question](#11-the-core-question)
    - [1.2 The Calibration Challenge](#12-the-calibration-challenge)
    - [1.3 Current Values Are STARTING POINTS](#13-current-values-are-starting-points)
  - [2. WHERE DO POINTS \& WEIGHTS COME FROM?](#2-where-do-points--weights-come-from)
    - [2.1 Three Sources of Scoring Values](#21-three-sources-of-scoring-values)
    - [2.2 Current Weight Derivation (Explained)](#22-current-weight-derivation-explained)
  - [3. DATA COLLECTION REQUIREMENTS](#3-data-collection-requirements)
    - [3.1 What Data Do You Need?](#31-what-data-do-you-need)
    - [3.2 Minimum Dataset Size](#32-minimum-dataset-size)
    - [3.3 Data Collection Methods](#33-data-collection-methods)
  - [4. STEP-BY-STEP CALIBRATION PROCESS](#4-step-by-step-calibration-process)
    - [4.1 The Complete Calibration Workflow](#41-the-complete-calibration-workflow)
    - [4.2 Python Code for Calibration Framework](#42-python-code-for-calibration-framework)
  - [5. INDICATOR 1: SIZE DISCREPANCY CALIBRATION](#5-indicator-1-size-discrepancy-calibration)
    - [5.1 Official Scoring (from Fraud Detection Logic)](#51-official-scoring-from-fraud-detection-logic)
    - [5.2 Where These Thresholds Come From](#52-where-these-thresholds-come-from)
    - [5.2 How to Find the Right Thresholds](#52-how-to-find-the-right-thresholds)
    - [5.3 Example with Real Numbers](#53-example-with-real-numbers)
    - [5.4 Visualization](#54-visualization)
  - [6. INDICATOR 2: CROP TYPE MISMATCH CALIBRATION](#6-indicator-2-crop-type-mismatch-calibration)
    - [6.1 Official Scoring (from Fraud Detection Logic)](#61-official-scoring-from-fraud-detection-logic)
    - [6.2 Where These Points Come From](#62-where-these-points-come-from)
    - [6.2 Crop Similarity Matrix](#62-crop-similarity-matrix)
    - [6.3 How to Calculate Crop Match Score](#63-how-to-calculate-crop-match-score)
  - [7. INDICATOR 3: WEATHER VALIDATION CALIBRATION](#7-indicator-3-weather-validation-calibration)
    - [7.1 Official Scoring (from Fraud Detection Logic)](#71-official-scoring-from-fraud-detection-logic)
    - [7.2 Crop Water Requirements (Literature-Based)](#72-crop-water-requirements-literature-based)
    - [7.3 Calibration with Regional Data](#73-calibration-with-regional-data)
  - [8. INDICATOR 4: GHOST FARMER CALIBRATION](#8-indicator-4-ghost-farmer-calibration)
    - [8.1 Official Scoring (from Fraud Detection Logic)](#81-official-scoring-from-fraud-detection-logic)
    - [8.2 Unit Clarification](#82-unit-clarification)
    - [8.2 Calibration with Regional Data](#82-calibration-with-regional-data)
    - [8.3 Regional Considerations](#83-regional-considerations)
  - [9. INDICATOR 5: HISTORICAL CONSISTENCY CALIBRATION](#9-indicator-5-historical-consistency-calibration)
    - [9.1 Official Scoring (from Fraud Detection Logic)](#91-official-scoring-from-fraud-detection-logic)
    - [9.2 Scientific Basis for NDVI Thresholds](#92-scientific-basis-for-ndvi-thresholds)
    - [9.3 Calibration Method](#93-calibration-method)
  - [10. INDICATOR 6: DISASTER CLAIM VALIDATION CALIBRATION](#10-indicator-6-disaster-claim-validation-calibration)
    - [10.1 Official Scoring (from Fraud Detection Logic)](#101-official-scoring-from-fraud-detection-logic)
    - [10.2 Flood Detection Calibration](#102-flood-detection-calibration)
    - [10.3 Drought Detection Calibration](#103-drought-detection-calibration)
  - [11. INDICATOR 7: CROPLAND SIGNAL CALIBRATION](#11-indicator-7-cropland-signal-calibration)
    - [11.1 Official Scoring (from Fraud Detection Logic)](#111-official-scoring-from-fraud-detection-logic)
    - [11.2 Calibration with Ground Truth](#112-calibration-with-ground-truth)
  - [12. WEIGHT ASSIGNMENT METHODOLOGY](#12-weight-assignment-methodology)
    - [12.1 The Weight Problem](#121-the-weight-problem)
    - [12.2 Data-Driven Weight Calculation](#122-data-driven-weight-calculation)
    - [12.3 Converting Weights to Points](#123-converting-weights-to-points)
    - [12.4 Alternative: AUC-Based Weights](#124-alternative-auc-based-weights)
  - [13. RISK THRESHOLD CALIBRATION](#13-risk-threshold-calibration)
    - [13.1 The Risk Threshold Problem](#131-the-risk-threshold-problem)
    - [13.2 Calibration Using Score Distribution](#132-calibration-using-score-distribution)
    - [13.3 Visualization of Thresholds](#133-visualization-of-thresholds)
    - [13.4 Balancing False Positives vs False Negatives](#134-balancing-false-positives-vs-false-negatives)
  - [14. VALIDATION \& TESTING PROTOCOL](#14-validation--testing-protocol)
    - [14.1 Validation Metrics](#141-validation-metrics)
    - [14.2 Validation Procedure](#142-validation-procedure)
    - [14.3 Cross-Validation](#143-cross-validation)
  - [15. CONTINUOUS IMPROVEMENT PROCESS](#15-continuous-improvement-process)
    - [15.1 Monitoring in Production](#151-monitoring-in-production)
    - [15.2 When to Recalibrate](#152-when-to-recalibrate)
    - [15.3 Recalibration Process](#153-recalibration-process)
  - [APPENDIX: COMPLETE CALIBRATION CODE](#appendix-complete-calibration-code)
    - [Full Python Implementation](#full-python-implementation)
  - [SUMMARY: COMPLETE SCORING PROCESS](#summary-complete-scoring-process)
    - [Step-by-Step How Points Are Calculated](#step-by-step-how-points-are-calculated)
    - [Example Calculation](#example-calculation)
    - [Official Weight Summary](#official-weight-summary)

---

## COMPLETE SCORING SUMMARY (Official Fraud Detection Logic)

### All 7 Indicators at a Glance

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MERIDIAN SENTINEL FRAUD SCORING SYSTEM                    │
│                         TOTAL POSSIBLE: 150 POINTS                           │
│                      SCALED SCORE: (Total/150) × 100                         │
└─────────────────────────────────────────────────────────────────────────────┘

INDICATOR 1: SIZE DISCREPANCY (30 points max)
├── 0-15% difference:   0 points  (normal variation)
├── 15-30% difference:  5 points  (minor discrepancy)
├── 30-50% difference: 15 points  (moderate discrepancy)
├── 50-70% difference: 25 points  (major discrepancy)
└── >70% difference:   30 points  (severe fraud indicator)

INDICATOR 2: CROP TYPE MISMATCH (30 points max)
├── Crops MATCH:                    0 points
├── Mismatch + High conf (>80%):   30 points
├── Mismatch + Med conf (65-80%):  20 points
└── Mismatch + Low conf (<65%):    10 points

INDICATOR 3: WEATHER VALIDATION (20 points max)
├── <50% required rainfall:   20 points (impossible)
├── 50-70% required rainfall: 15 points (marginal)
├── 70-85% required rainfall:  8 points (stressed)
└── >85% required rainfall:    0 points (adequate)

INDICATOR 4: GHOST FARMER DETECTION (20 points max)
├── <1 person/hectare:   20 points (uninhabited)
├── 1-5 people/hectare:  10 points (very sparse)
└── >5 people/hectare:    0 points (populated)

INDICATOR 5: HISTORICAL CONSISTENCY (15 points max + 15 bonus)
├── Major change (>0.4):        15 points
├── Moderate change (0.2-0.4):  10 points
├── Stable (<0.2):               0 points
└── BONUS: Forest→Cropland:    +15 points

INDICATOR 6: DISASTER CLAIM VALIDATION (10 points max)
├── No disaster claim filed:     0 points
├── Disaster CONFIRMED:          0 points
└── NO evidence of disaster:    10 points

INDICATOR 7: CROPLAND SIGNAL (10 points max)
├── Cropland prob <30% AND NDVI <0.35: 10 points
└── Otherwise:                          0 points

───────────────────────────────────────────────────────────────────────────────
TOTAL MAXIMUM: 150 points (165 with forest bonus)
SCALED SCORE = (Total / 150) × 100 = 0-100 final score
───────────────────────────────────────────────────────────────────────────────

RISK LEVELS:
├── 0-39 points:   LOW RISK (Green)    → AUTO-APPROVE
├── 40-69 points:  MEDIUM RISK (Yellow) → MANUAL REVIEW
└── 70-100 points: HIGH RISK (Red)     → REJECT/INVESTIGATE
```

### Performance Targets

| Metric              | Target | Meaning                            |
| ------------------- | ------ | ---------------------------------- |
| True Positive Rate  | > 85%  | Catch 85%+ of actual fraud         |
| False Positive Rate | < 10%  | Don't block legitimate farmers     |
| Precision           | > 85%  | When flagging fraud, be right 85%+ |
| F1 Score            | > 85%  | Balanced performance               |
| Processing Time     | < 12s  | Real-time verification             |

---

## 1. INTRODUCTION: UNDERSTANDING THE SCORING PROBLEM

### 1.1 The Core Question

**Your Question:** "How do I get these points and weights? Where do they come from?"

**Answer:** The points and weights are derived from:

1. **Official Specification** - The Fraud Detection Logic document defines the exact values
2. **Domain Expert Judgment** - Agricultural fraud patterns in Sub-Saharan Africa
3. **Scientific Literature** - FAO crop requirements, remote sensing thresholds
4. **Data Calibration** - Values refined with real labeled data (500+ farms)

**IMPORTANT:** Production deployment requires validation with REAL labeled data!

### 1.2 The Calibration Challenge

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THE CALIBRATION CHALLENGE                         │
└─────────────────────────────────────────────────────────────────────┘

PROBLEM: We have 7 indicators, each with:
├── Multiple thresholds (e.g., 15%, 30%, 50% for size)
├── Point assignments (e.g., 0, 10, 20, 30 points)
└── Relative weights (e.g., 22.2% vs 7.4%)

QUESTION: How do we know these are correct?

ANSWER: We must CALIBRATE using:
├── Ground truth data (farms with KNOWN fraud status)
├── Statistical analysis (ROC curves, precision-recall)
├── Iterative optimization (adjust until metrics are optimal)
└── Continuous monitoring (refine over time)
```

### 1.3 Current Values Are STARTING POINTS

| What We Have       | What We Need                     |
| ------------------ | -------------------------------- |
| Initial estimates  | Calibrated values                |
| Expert judgment    | Data-driven thresholds           |
| Logical weights    | Statistically optimized weights  |
| Assumed thresholds | Empirically validated thresholds |

---

## 2. WHERE DO POINTS & WEIGHTS COME FROM?

### 2.1 Three Sources of Scoring Values

```
┌─────────────────────────────────────────────────────────────────────┐
│         THREE SOURCES OF SCORING VALUES                              │
└─────────────────────────────────────────────────────────────────────┘

SOURCE 1: DOMAIN EXPERT JUDGMENT (Current State)
├── Agricultural experts estimate fraud patterns
├── Insurance investigators share experience
├── Based on "what usually indicates fraud"
├── LIMITATION: Subjective, may not generalize

SOURCE 2: STATISTICAL ANALYSIS (Required)
├── Analyze real fraud cases vs legitimate cases
├── Calculate optimal thresholds using ROC curves
├── Determine weights using logistic regression
├── ADVANTAGE: Data-driven, objective

SOURCE 3: MACHINE LEARNING (Advanced)
├── Train models on labeled data
├── Learn optimal weights automatically
├── Continuously adapt to new patterns
├── ADVANTAGE: Adaptive, high accuracy
```

### 2.2 Current Weight Derivation (Explained)

**WHY Size Discrepancy = 30 points (22.2%)?**

```
REASONING:
├── Size is DIRECTLY measurable from satellite
├── Claiming 10 hectares but having 2 hectares = CLEAR fraud
├── High reliability = High weight
├── Historical fraud cases show size inflation is COMMON
└── RESULT: Highest weight (30 points)
```

**WHY Cropland Signal = 10 points (7.4%)?**

```
REASONING:
├── Land cover classification has some uncertainty
├── Edge cases exist (transitional land)
├── Not conclusive on its own
├── LOWER reliability = LOWER weight
└── RESULT: Lowest weight (10 points)
```

---

## 3. DATA COLLECTION REQUIREMENTS

### 3.1 What Data Do You Need?

To calibrate the scoring system, you need a **LABELED DATASET** with:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REQUIRED LABELED DATASET                          │
└─────────────────────────────────────────────────────────────────────┘

FOR EACH FARM IN YOUR DATASET:

1. FARMER APPLICATION DATA:
   ├── Claimed location (lat, lon)
   ├── Claimed area (hectares)
   ├── Claimed crop type
   ├── Planting date
   └── Any disaster claims

2. GROUND TRUTH DATA:
   ├── ACTUAL area (GPS-surveyed)
   ├── ACTUAL crop type (field verified)
   ├── ACTUAL planting date
   └── FRAUD STATUS: [LEGITIMATE | FRAUD]  ← CRITICAL!

3. SATELLITE-DERIVED VALUES:
   ├── Detected area from GEE
   ├── NDVI values
   ├── Cropland probability
   └── All 7 indicator raw values
```

### 3.2 Minimum Dataset Size

| Dataset Component     | Minimum Size | Recommended Size |
| --------------------- | ------------ | ---------------- |
| Total farms           | 500          | 2,000+           |
| Confirmed fraud cases | 50           | 200+             |
| Legitimate farms      | 450          | 1,800+           |
| Geographic regions    | 3            | 5+               |
| Crop types            | 5            | 10+              |

### 3.3 Data Collection Methods

```
METHOD 1: PARTNER WITH INSURANCE COMPANIES
├── Access historical claims data
├── Get fraud investigation outcomes
└── ~60% of fraud cases are detected post-payout

METHOD 2: PARTNER WITH GOVERNMENT AGENCIES
├── Access subsidy application records
├── Get audit results
└── Field verification records

METHOD 3: CONDUCT FIELD SURVEYS
├── Sample farms from satellite detection
├── GPS survey actual boundaries
├── Verify crop types on ground
└── Document discrepancies

METHOD 4: EXPERT LABELING
├── Show satellite imagery to agronomists
├── Have them classify fraud likelihood
├── Use consensus among 3+ experts
```

---

## 4. STEP-BY-STEP CALIBRATION PROCESS

### 4.1 The Complete Calibration Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│           COMPLETE CALIBRATION WORKFLOW                              │
└─────────────────────────────────────────────────────────────────────┘

PHASE 1: DATA PREPARATION (Weeks 1-4)
├── Step 1.1: Collect labeled dataset (500+ farms)
├── Step 1.2: Split into TRAIN (70%) and TEST (30%)
├── Step 1.3: Calculate all 7 indicator raw values for each farm
└── Step 1.4: Verify data quality and completeness

PHASE 2: THRESHOLD CALIBRATION (Weeks 5-6)
├── Step 2.1: For each indicator, plot distribution
├── Step 2.2: Compare fraud vs legitimate distributions
├── Step 2.3: Find optimal thresholds using ROC curves
└── Step 2.4: Validate thresholds on test set

PHASE 3: WEIGHT OPTIMIZATION (Weeks 7-8)
├── Step 3.1: Use logistic regression to find optimal weights
├── Step 3.2: Alternatively, use grid search optimization
├── Step 3.3: Validate that high-weight indicators are predictive
└── Step 3.4: Adjust weights based on business priorities

PHASE 4: RISK LEVEL CALIBRATION (Week 9)
├── Step 4.1: Plot score distribution for fraud vs legitimate
├── Step 4.2: Choose LOW/MEDIUM/HIGH thresholds
├── Step 4.3: Balance false positives vs false negatives
└── Step 4.4: Validate on test set

PHASE 5: VALIDATION (Week 10)
├── Step 5.1: Calculate F1 score, precision, recall
├── Step 5.2: Ensure FP rate < 10%, FN rate < 5%
├── Step 5.3: Manual review of edge cases
└── Step 5.4: Document final calibrated values
```

### 4.2 Python Code for Calibration Framework

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_curve, auc, precision_recall_curve
from sklearn.linear_model import LogisticRegression
import matplotlib.pyplot as plt

class FraudScoringCalibrator:
    """
    Calibrates fraud detection thresholds and weights using labeled data.
    """

    def __init__(self, data_path):
        """
        Initialize with path to labeled dataset CSV.

        Expected columns:
        - farm_id: Unique identifier
        - is_fraud: 1 = fraud, 0 = legitimate
        - size_discrepancy_pct: Percentage difference in area
        - crop_match_score: 0-1 similarity score
        - rainfall_deficit_pct: Rainfall deficit percentage
        - population_density: People per km²
        - ndvi_change: Historical NDVI change
        - disaster_confirmed: 1 = confirmed, 0 = not
        - cropland_probability: 0-1 probability
        """
        self.df = pd.read_csv(data_path)
        self.train_df = None
        self.test_df = None
        self.calibrated_thresholds = {}
        self.calibrated_weights = {}

    def prepare_data(self, test_size=0.3, random_state=42):
        """Split data into train and test sets."""
        self.train_df, self.test_df = train_test_split(
            self.df,
            test_size=test_size,
            random_state=random_state,
            stratify=self.df['is_fraud']
        )
        print(f"Training set: {len(self.train_df)} farms")
        print(f"Test set: {len(self.test_df)} farms")
        print(f"Fraud rate: {self.df['is_fraud'].mean():.1%}")

    def find_optimal_threshold(self, indicator_column, indicator_name):
        """
        Find optimal threshold for a single indicator using ROC curve.
        Returns threshold that maximizes Youden's J statistic.
        """
        # Get values for fraud and legitimate farms
        fraud = self.train_df[self.train_df['is_fraud'] == 1][indicator_column]
        legit = self.train_df[self.train_df['is_fraud'] == 0][indicator_column]

        # Calculate ROC curve
        y_true = self.train_df['is_fraud']
        y_scores = self.train_df[indicator_column]

        fpr, tpr, thresholds = roc_curve(y_true, y_scores)

        # Find optimal threshold (Youden's J = TPR - FPR)
        j_scores = tpr - fpr
        optimal_idx = np.argmax(j_scores)
        optimal_threshold = thresholds[optimal_idx]

        # Calculate AUC
        roc_auc = auc(fpr, tpr)

        print(f"\n{indicator_name}:")
        print(f"  AUC: {roc_auc:.3f}")
        print(f"  Optimal threshold: {optimal_threshold:.3f}")
        print(f"  At this threshold: TPR={tpr[optimal_idx]:.2f}, FPR={fpr[optimal_idx]:.2f}")

        return {
            'threshold': optimal_threshold,
            'auc': roc_auc,
            'tpr': tpr[optimal_idx],
            'fpr': fpr[optimal_idx]
        }
```

---

## 5. INDICATOR 1: SIZE DISCREPANCY CALIBRATION

### 5.1 Official Scoring (from Fraud Detection Logic)

```
INDICATOR 1: SIZE DISCREPANCY
Weight: 30 points (Maximum)

FORMULA:
Difference (%) = |Detected Area - Claimed Area| / Claimed Area × 100

OFFICIAL POINT ALLOCATION:
├── 0-15% difference:   0 points  (normal variation)
├── 15-30% difference:  5 points  (minor discrepancy)
├── 30-50% difference:  15 points (moderate discrepancy)
├── 50-70% difference:  25 points (major discrepancy)
└── >70% difference:    30 points (severe fraud indicator)

RATIONALE:
├── Farmers often inflate farm sizes for larger payouts
├── Satellite imagery provides objective measurement
└── 15% tolerance accounts for measurement uncertainty
```

### 5.2 Where These Thresholds Come From

```
THRESHOLD DERIVATION:

1. 15% TOLERANCE (0 points)
   ├── GPS measurement error: ±3-5%
   ├── Irregular boundary interpretation: ±5-8%
   ├── Seasonal boundary variation: ±2-3%
   └── TOTAL reasonable error: ~15%

2. 30% THRESHOLD (5 points)
   └── Beyond normal error but not definitive fraud

3. 50% THRESHOLD (15 points)
   └── Claiming 1 hectare but having 0.5 hectares (50% inflation)

4. 70% THRESHOLD (25 points)
   └── Claiming 1 hectare but having 0.3 hectares (serious over-claim)

5. >70% THRESHOLD (30 points)
   └── Claiming more than 3x actual size (likely fraud)
```

### 5.2 How to Find the Right Thresholds

```python
def calibrate_size_discrepancy(self):
    """
    Calibrate size discrepancy thresholds using real data.
    """
    # Step 1: Calculate size discrepancy for each farm
    # size_discrepancy_pct = abs(claimed_area - detected_area) / claimed_area * 100

    # Step 2: Analyze distribution
    fraud_disc = self.train_df[self.train_df['is_fraud']==1]['size_discrepancy_pct']
    legit_disc = self.train_df[self.train_df['is_fraud']==0]['size_discrepancy_pct']

    print("Size Discrepancy Distribution:")
    print(f"  Legitimate farms - Mean: {legit_disc.mean():.1f}%, Median: {legit_disc.median():.1f}%")
    print(f"  Fraud farms - Mean: {fraud_disc.mean():.1f}%, Median: {fraud_disc.median():.1f}%")

    # Step 3: Find optimal thresholds using percentiles
    # Threshold 1: 95th percentile of legitimate farms
    threshold_1 = np.percentile(legit_disc, 95)

    # Threshold 2: Median of fraud farms
    threshold_2 = np.percentile(fraud_disc, 50)

    # Threshold 3: 75th percentile of fraud farms
    threshold_3 = np.percentile(fraud_disc, 75)

    print(f"\nCalibrated Thresholds:")
    print(f"  Threshold 1 (95% legit): {threshold_1:.1f}%")
    print(f"  Threshold 2 (50% fraud): {threshold_2:.1f}%")
    print(f"  Threshold 3 (75% fraud): {threshold_3:.1f}%")

    return {
        'threshold_1': threshold_1,  # Below this = 0 points
        'threshold_2': threshold_2,  # Below this = 10 points
        'threshold_3': threshold_3   # Below this = 20 points, above = 30 points
    }
```

### 5.3 Example with Real Numbers

```
EXAMPLE: Suppose you collect data from 500 farms:
├── 450 legitimate farms
└── 50 confirmed fraud cases

ANALYSIS RESULTS:
┌────────────────────────────────────────────────────────────────┐
│ Legitimate Farms Size Discrepancy:                             │
│   Mean: 8.2%    Median: 6.5%    95th percentile: 18.3%        │
├────────────────────────────────────────────────────────────────┤
│ Fraud Farms Size Discrepancy:                                  │
│   Mean: 67.4%   Median: 52.1%   25th percentile: 35.2%        │
└────────────────────────────────────────────────────────────────┘

CALIBRATED THRESHOLDS:
├── 0-18%:  0 points  (covers 95% of legitimate farms)
├── 18-35%: 10 points (suspicious zone)
├── 35-52%: 20 points (likely fraud - median of fraud)
└── >52%:   30 points (very likely fraud)

COMPARE TO ORIGINAL:
├── Original: 15%, 30%, 50%
└── Calibrated: 18%, 35%, 52%
```

### 5.4 Visualization

```
┌─────────────────────────────────────────────────────────────────────┐
│    SIZE DISCREPANCY DISTRIBUTION                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    Legitimate Farms          Fraud Farms                            │
│    ████████████              ░░░░                                   │
│    ████████████████          ░░░░░░░░                               │
│    ████████████████████      ░░░░░░░░░░░░                           │
│    ███████████████████████   ░░░░░░░░░░░░░░░░                       │
│    █████████████████████████ ░░░░░░░░░░░░░░░░░░░░░░░░               │
│    ├──────────┼──────────┼──────────┼──────────┼────────            │
│    0%        18%        35%        52%       100%                   │
│              ↑          ↑          ↑                                │
│         Threshold1  Threshold2  Threshold3                          │
│         (0→10pts)  (10→20pts)  (20→30pts)                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. INDICATOR 2: CROP TYPE MISMATCH CALIBRATION

### 6.1 Official Scoring (from Fraud Detection Logic)

```
INDICATOR 2: CROP TYPE MISMATCH
Weight: 30 points (Maximum)

LOGIC: Compare claimed crop type against satellite-detected crop
       using spectral analysis (NDVI signatures)

OFFICIAL POINT ALLOCATION:
├── Crops MATCH:     0 points
├── Crops DON'T MATCH:
│   ├── High confidence (>80%):   30 points
│   ├── Medium confidence (65-80%): 20 points
│   └── Low confidence (<65%):    10 points

RATIONALE:
├── Farmers may claim high-value crops (e.g., rice)
│   while growing low-value crops (e.g., cassava)
├── Spectral signatures differ between crop types
└── Use confidence thresholds to reduce false positives
```

### 6.2 Where These Points Come From

```
CONFIDENCE LEVEL EXPLANATION:

1. HIGH CONFIDENCE (>80% detection accuracy)
   ├── Clear NDVI signature match to specific crop
   ├── Multiple satellite passes confirm detection
   └── RESULT: Full 30 points (strong evidence of mismatch)

2. MEDIUM CONFIDENCE (65-80%)
   ├── Likely crop detection but some uncertainty
   ├── Could be crop rotation or mixed planting
   └── RESULT: 20 points (probable mismatch)

3. LOW CONFIDENCE (<65%)
   ├── Weak signal, clouds, or ambiguous signature
   ├── Could be legitimate or detection error
   └── RESULT: 10 points (possible mismatch)

4. CROPS MATCH
   └── RESULT: 0 points (no discrepancy)
```

### 6.2 Crop Similarity Matrix

You need to build a **CROP SIMILARITY MATRIX** based on NDVI signatures:

```python
def build_crop_similarity_matrix(self):
    """
    Build crop similarity matrix from real NDVI profile data.
    """
    # Crops grouped by similar NDVI signatures
    crop_families = {
        'cereals': ['maize', 'sorghum', 'millet', 'wheat', 'rice'],
        'legumes': ['beans', 'groundnuts', 'soybeans', 'cowpeas'],
        'roots': ['cassava', 'sweet_potato', 'yam', 'potato'],
        'cash_crops': ['cotton', 'tobacco', 'sugarcane'],
        'vegetables': ['tomato', 'onion', 'cabbage', 'pepper']
    }

    # Similarity scoring:
    # Same crop = 1.0 (0 points)
    # Same family = 0.7 (15 points)
    # Different family = 0.0 (30 points)

    similarity_matrix = {}
    for family, crops in crop_families.items():
        for crop1 in crops:
            similarity_matrix[crop1] = {}
            for family2, crops2 in crop_families.items():
                for crop2 in crops2:
                    if crop1 == crop2:
                        similarity_matrix[crop1][crop2] = 1.0
                    elif family == family2:
                        similarity_matrix[crop1][crop2] = 0.7
                    else:
                        similarity_matrix[crop1][crop2] = 0.0

    return similarity_matrix
```

### 6.3 How to Calculate Crop Match Score

```
STEP 1: Get claimed crop from farmer application
        claimed_crop = "maize"

STEP 2: Detect crop from satellite NDVI signature
        detected_crop = classify_crop_from_ndvi(farm_boundary)
        detected_crop = "sorghum"

STEP 3: Look up similarity
        similarity = similarity_matrix["maize"]["sorghum"]
        similarity = 0.7 (both cereals)

STEP 4: Calculate score
        IF similarity == 1.0: score = 0 points (exact match)
        IF 0.5 <= similarity < 1.0: score = 15 points (similar)
        IF similarity < 0.5: score = 30 points (different)

        RESULT: 15 points
```

---

## 7. INDICATOR 3: WEATHER VALIDATION CALIBRATION

### 7.1 Official Scoring (from Fraud Detection Logic)

```
INDICATOR 3: WEATHER VALIDATION
Weight: 20 points (Maximum)

LOGIC: Check if local rainfall was sufficient for claimed crop to survive

CALCULATION:
├── Get 6-month rainfall from planting date
├── Compare to crop-specific minimum requirements

CROP MINIMUM REQUIREMENTS (from official spec):
├── Maize:   450mm (6-month period)
├── Rice:    800mm
├── Cassava: 300mm
├── Sorghum: 400mm

OFFICIAL POINT ALLOCATION:
├── <50% of required rainfall:    20 points (impossible conditions)
├── 50-70% of required rainfall:  15 points (marginal conditions)
├── 70-85% of required rainfall:   8 points (stressed conditions)
└── >85% of required rainfall:     0 points (adequate rainfall)

RATIONALE:
├── Crops cannot survive without adequate water
├── Claims in drought areas are suspicious
└── 6-month cumulative rainfall used for accuracy
```

### 7.2 Crop Water Requirements (Literature-Based)

| Crop       | Water Requirement (mm/season) | Source                             |
| ---------- | ----------------------------- | ---------------------------------- |
| Maize      | 500-800                       | FAO Irrigation & Drainage Paper 56 |
| Wheat      | 450-650                       | FAO                                |
| Rice       | 900-2000                      | FAO                                |
| Sorghum    | 450-650                       | FAO                                |
| Cotton     | 700-1300                      | FAO                                |
| Beans      | 300-500                       | FAO                                |
| Groundnuts | 500-700                       | FAO                                |
| Cassava    | 1000-1500                     | FAO                                |

### 7.3 Calibration with Regional Data

```python
def calibrate_weather_validation(self, crop_type, region):
    """
    Calibrate rainfall thresholds for specific crop and region.
    """
    # Step 1: Get crop water requirement (from FAO or local research)
    crop_requirements = {
        'maize': {'min': 500, 'optimal': 650, 'max': 800},
        'sorghum': {'min': 450, 'optimal': 550, 'max': 650},
        'beans': {'min': 300, 'optimal': 400, 'max': 500}
    }

    req = crop_requirements.get(crop_type, {'min': 500, 'optimal': 600, 'max': 700})

    # Step 2: Analyze actual yield data vs rainfall
    # This requires yield data paired with rainfall data

    # Step 3: Set thresholds based on yield response
    thresholds = {
        'adequate': req['min'],        # Above this = 0 points
        'mild_deficit': req['min'] * 0.6,  # Above this = 10 points
        'severe_deficit': 0           # Below mild = 20 points
    }

    return thresholds
```

---

## 8. INDICATOR 4: GHOST FARMER CALIBRATION

### 8.1 Official Scoring (from Fraud Detection Logic)

```
INDICATOR 4: GHOST FARMER DETECTION
Weight: 20 points (Maximum)

LOGIC: Check population density around farm
       Uninhabited areas cannot have active farmers

CALCULATION:
├── Get population density within 1km radius
├── Based on WorldPop dataset

OFFICIAL POINT ALLOCATION:
├── <1 person/hectare:    20 points (uninhabited)
├── 1-5 people/hectare:   10 points (very sparse)
└── >5 people/hectare:     0 points (populated)

RATIONALE:
├── Fraudsters create fake farms in remote/uninhabited areas
├── Real farmers live near their farms
└── 1km radius accounts for distance between home and farm
```

### 8.2 Unit Clarification

```
IMPORTANT: Conversion between units

FROM SPEC:
├── <1 person/hectare = <100 people/km²
├── 1-5 people/hectare = 100-500 people/km²
└── >5 people/hectare = >500 people/km²

WorldPop provides data in people/km², so:
├── <100/km²:    20 points (uninhabited)
├── 100-500/km²: 10 points (sparse)
└── >500/km²:     0 points (populated)
```

### 8.2 Calibration with Regional Data

```python
def calibrate_ghost_farmer(self, region):
    """
    Calibrate population density thresholds for ghost farmer detection.
    """
    # Step 1: Analyze population density distribution in farming areas
    farm_pop_density = self.train_df[self.train_df['is_farm'] == 1]['population_density']

    # Step 2: Find natural breaks in the data
    # Use Jenks natural breaks or percentiles
    low_threshold = np.percentile(farm_pop_density, 5)   # 5th percentile
    mid_threshold = np.percentile(farm_pop_density, 15)  # 15th percentile

    print(f"Population Density in Farming Areas:")
    print(f"  5th percentile: {low_threshold:.1f}/km²")
    print(f"  15th percentile: {mid_threshold:.1f}/km²")
    print(f"  Median: {np.median(farm_pop_density):.1f}/km²")

    # Step 3: Cross-reference with fraud cases
    fraud_pop = self.train_df[self.train_df['is_fraud']==1]['population_density']
    legit_pop = self.train_df[self.train_df['is_fraud']==0]['population_density']

    print(f"\nFraud farms population density: Mean={fraud_pop.mean():.1f}/km²")
    print(f"Legit farms population density: Mean={legit_pop.mean():.1f}/km²")

    return {
        'high_risk': low_threshold,    # Below = 20 points
        'medium_risk': mid_threshold   # Below = 10 points, above = 0 points
    }
```

### 8.3 Regional Considerations

```
IMPORTANT: Population density thresholds vary by region!

KENYA:
├── Dense farming areas: > 200/km²
├── Normal farming areas: 50-200/km²
└── Sparse areas: < 50/km²

MALI (Sahel):
├── Dense farming areas: > 50/km²
├── Normal farming areas: 10-50/km²
└── Sparse areas: < 10/km²

RECOMMENDATION: Calibrate thresholds PER COUNTRY or PER REGION
```

---

## 9. INDICATOR 5: HISTORICAL CONSISTENCY CALIBRATION

### 9.1 Official Scoring (from Fraud Detection Logic)

```
INDICATOR 5: HISTORICAL CONSISTENCY
Weight: 15 points (Maximum)
BONUS: +15 points for forest-to-cropland conversion (Total possible: 30)

LOGIC: Check for recent land use changes indicating new farm creation

CALCULATION:
├── Compare NDVI between current year and 5 years ago
├── NDVI Change = |Current NDVI - Historical NDVI|

OFFICIAL POINT ALLOCATION:
├── Major change (>0.4):      15 points (recent conversion)
├── Moderate change (0.2-0.4): 10 points (some change)
└── Stable (<0.2):             0 points (consistent farmland)

ADDITIONAL PENALTY:
└── Forest to cropland conversion: +15 points (deforestation fraud)

RATIONALE:
├── Fraudsters may create farms on recently cleared land
├── Sudden changes are suspicious
└── Moderate changes allowed for crop rotation
```

### 9.2 Scientific Basis for NDVI Thresholds

```
NDVI VALUES REFERENCE:
├── 0.0 - 0.1:  Bare soil, rocks, urban
├── 0.1 - 0.2:  Sparse vegetation, stressed crops
├── 0.2 - 0.4:  Moderate vegetation, early/late season crops
├── 0.4 - 0.6:  Dense vegetation, healthy crops
└── 0.6 - 0.9:  Very dense vegetation, forest

INTERPRETATION OF CHANGE:
├── Change < 0.1:  Seasonal variation (normal)
├── Change 0.1-0.2: Crop rotation or weather variation
├── Change 0.2-0.3: Significant change, possible land conversion
└── Change > 0.3:  Major land use change (forest→farm, urban→farm)
```

### 9.3 Calibration Method

```python
def calibrate_historical_consistency(self):
    """
    Calibrate NDVI change thresholds.
    """
    # Get NDVI changes for fraud vs legitimate
    fraud_change = self.train_df[self.train_df['is_fraud']==1]['ndvi_change']
    legit_change = self.train_df[self.train_df['is_fraud']==0]['ndvi_change']

    print("NDVI Change Distribution:")
    print(f"  Legitimate: Mean={legit_change.mean():.3f}, 95th pct={np.percentile(legit_change, 95):.3f}")
    print(f"  Fraud: Mean={fraud_change.mean():.3f}, 25th pct={np.percentile(fraud_change, 25):.3f}")

    # Thresholds based on distribution
    return {
        'stable': np.percentile(legit_change, 95),      # 95% of legit farms below this
        'suspicious': np.percentile(fraud_change, 25)   # 25% of fraud farms below this
    }
```

---

## 10. INDICATOR 6: DISASTER CLAIM VALIDATION CALIBRATION

### 10.1 Official Scoring (from Fraud Detection Logic)

```
INDICATOR 6: DISASTER CLAIM VALIDATION
Weight: 10 points (Maximum)

LOGIC: For damage claims (flood, drought), verify using satellite evidence

CALCULATION FOR FLOOD CLAIMS:
├── Use Sentinel-1 SAR to detect water
├── Check Global Surface Water for flood-prone areas
└── If NO flood detected: 10 points

CALCULATION FOR DROUGHT CLAIMS:
├── Compare rainfall to historical average
└── If rainfall >70% of average: 10 points (no drought)

OFFICIAL POINT ALLOCATION:
├── No disaster claim filed: 0 points (not applicable)
├── Disaster CONFIRMED by satellite: 0 points
└── NO evidence of claimed disaster: 10 points

RATIONALE:
├── Fraudsters file false disaster claims for compensation
├── Satellite data can verify actual disaster occurrence
└── Only applies to farms with active disaster claims
```

### 10.2 Flood Detection Calibration

```python
def calibrate_flood_detection(self):
    """
    Calibrate VV backscatter change thresholds for flood detection.
    """
    # Research-based thresholds (from Sentinel-1 flood detection literature)

    # VV backscatter DECREASES during flooding (water is smooth)
    # Typical values:
    # - Normal land: -8 to -12 dB
    # - Flooded land: -15 to -25 dB

    thresholds = {
        'definitely_flooded': -5,    # VV change < -5 dB = confirmed flood
        'possibly_flooded': -3,      # VV change -3 to -5 dB = partial evidence
        'not_flooded': -3            # VV change > -3 dB = no flood
    }

    return thresholds
```

### 10.3 Drought Detection Calibration

```
DROUGHT DEFINITION (Based on SPI - Standardized Precipitation Index):

├── Normal: Rainfall 80-120% of average
├── Mild drought: Rainfall 60-80% of average
├── Moderate drought: Rainfall 40-60% of average
├── Severe drought: Rainfall 20-40% of average
└── Extreme drought: Rainfall < 20% of average

SCORING:
├── Rainfall < 40% of average: Disaster CONFIRMED (0 points)
├── Rainfall 40-60% of average: PARTIAL evidence (5 points)
└── Rainfall > 60% of average: NO drought evidence (10 points)
```

---

## 11. INDICATOR 7: CROPLAND SIGNAL CALIBRATION

### 11.1 Official Scoring (from Fraud Detection Logic)

```
INDICATOR 7: CROPLAND SIGNAL
Weight: 10 points (Maximum)

LOGIC: Confirm location shows actual cropland signal
       (not forest, water, urban, etc.)

CALCULATION:
├── Check Dynamic World cropland probability
├── Check NDVI value

OFFICIAL POINT ALLOCATION:
├── Cropland probability <30% AND NDVI <0.35: 10 points
└── Otherwise: 0 points

RATIONALE:
├── Farm coordinates must show actual cropland
├── Non-agricultural land suggests fraud
└── Uses TWO independent signals for reliability:
    ├── Dynamic World land classification
    └── NDVI vegetation index
```

### 11.2 Calibration with Ground Truth

```python
def calibrate_cropland_signal(self):
    """
    Calibrate cropland probability thresholds.
    """
    # Step 1: Get Dynamic World cropland probability for known farms
    known_farms = self.train_df[self.train_df['is_actual_cropland']==1]
    non_farms = self.train_df[self.train_df['is_actual_cropland']==0]

    farm_prob = known_farms['cropland_probability']
    nonfarm_prob = non_farms['cropland_probability']

    print("Cropland Probability Distribution:")
    print(f"  Actual farms: Mean={farm_prob.mean():.2f}, 5th pct={np.percentile(farm_prob, 5):.2f}")
    print(f"  Non-farms: Mean={nonfarm_prob.mean():.2f}, 95th pct={np.percentile(nonfarm_prob, 95):.2f}")

    # Step 2: Find optimal thresholds
    high_threshold = np.percentile(farm_prob, 5)    # 95% of farms above this
    low_threshold = np.percentile(nonfarm_prob, 95) # 95% of non-farms below this

    return {
        'confirmed_cropland': high_threshold,   # Above = 0 points
        'possible_cropland': low_threshold      # Below = 10 points, between = 5 points
    }
```

---

## 12. WEIGHT ASSIGNMENT METHODOLOGY

### 12.1 The Weight Problem

```
QUESTION: Why is Size Discrepancy 30 points but Cropland Signal only 10 points?

ANSWER: Weights reflect:
├── ACCURACY: How reliably does this indicator predict fraud?
├── SPECIFICITY: How unique is this signal to fraud cases?
├── MEASURABILITY: How accurately can we measure this?
└── IMPORTANCE: How critical is this factor for decision-making?
```

### 12.2 Data-Driven Weight Calculation

```python
def calculate_optimal_weights(self):
    """
    Use logistic regression to find optimal indicator weights.
    """
    # Prepare features (all 7 indicators)
    X = self.train_df[[
        'size_discrepancy_score',
        'crop_mismatch_score',
        'weather_validation_score',
        'ghost_farmer_score',
        'historical_consistency_score',
        'disaster_validation_score',
        'cropland_signal_score'
    ]]
    y = self.train_df['is_fraud']

    # Fit logistic regression
    model = LogisticRegression(class_weight='balanced')
    model.fit(X, y)

    # Get coefficients (weights)
    weights = model.coef_[0]

    # Normalize to percentages
    abs_weights = np.abs(weights)
    normalized_weights = abs_weights / abs_weights.sum() * 100

    indicators = [
        'Size Discrepancy',
        'Crop Mismatch',
        'Weather Validation',
        'Ghost Farmer',
        'Historical Consistency',
        'Disaster Validation',
        'Cropland Signal'
    ]

    print("Optimal Weights (Data-Driven):")
    for name, weight in zip(indicators, normalized_weights):
        print(f"  {name}: {weight:.1f}%")

    return dict(zip(indicators, normalized_weights))
```

### 12.3 Converting Weights to Points

```
EXAMPLE OUTPUT FROM LOGISTIC REGRESSION:

Data-Driven Weights:
├── Size Discrepancy:       28.5%
├── Crop Mismatch:          24.2%
├── Weather Validation:     16.8%
├── Ghost Farmer:           12.3%
├── Historical Consistency:  9.4%
├── Disaster Validation:     5.1%
└── Cropland Signal:         3.7%

CONVERT TO POINT SCALE (Total = 100 points):
├── Size Discrepancy:       29 points (round 28.5)
├── Crop Mismatch:          24 points
├── Weather Validation:     17 points
├── Ghost Farmer:           12 points
├── Historical Consistency:  9 points
├── Disaster Validation:     5 points
└── Cropland Signal:         4 points
                           ────────
                TOTAL:     100 points
```

### 12.4 Alternative: AUC-Based Weights

```python
def calculate_auc_based_weights(self):
    """
    Assign weights based on individual indicator AUC scores.
    Higher AUC = higher predictive power = higher weight.
    """
    indicators = {
        'size_discrepancy_score': 'Size Discrepancy',
        'crop_mismatch_score': 'Crop Mismatch',
        'weather_validation_score': 'Weather Validation',
        'ghost_farmer_score': 'Ghost Farmer',
        'historical_consistency_score': 'Historical Consistency',
        'disaster_validation_score': 'Disaster Validation',
        'cropland_signal_score': 'Cropland Signal'
    }

    aucs = {}
    for col, name in indicators.items():
        result = self.find_optimal_threshold(col, name)
        aucs[name] = result['auc']

    # Normalize AUCs to weights
    total_auc = sum(aucs.values())
    weights = {k: v/total_auc * 100 for k, v in aucs.items()}

    return weights
```

---

## 13. RISK THRESHOLD CALIBRATION

### 13.1 The Risk Threshold Problem

```
CURRENT THRESHOLDS:
├── 0-39:   LOW RISK    → APPROVE
├── 40-69:  MEDIUM RISK → MANUAL REVIEW
└── 70-100: HIGH RISK   → REJECT

QUESTION: How do we know 40 and 70 are the right cutoffs?
```

### 13.2 Calibration Using Score Distribution

```python
def calibrate_risk_thresholds(self):
    """
    Find optimal risk thresholds based on score distributions.
    """
    # Calculate total fraud score for each farm
    fraud_scores = self.train_df[self.train_df['is_fraud']==1]['total_score']
    legit_scores = self.train_df[self.train_df['is_fraud']==0]['total_score']

    print("Score Distribution:")
    print(f"  Legitimate: Mean={legit_scores.mean():.1f}, Median={legit_scores.median():.1f}")
    print(f"  Fraud: Mean={fraud_scores.mean():.1f}, Median={fraud_scores.median():.1f}")

    # Find optimal thresholds
    # LOW/MEDIUM boundary: 95th percentile of legitimate farms
    low_medium_threshold = np.percentile(legit_scores, 95)

    # MEDIUM/HIGH boundary: 25th percentile of fraud farms
    medium_high_threshold = np.percentile(fraud_scores, 25)

    print(f"\nCalibrated Thresholds:")
    print(f"  LOW/MEDIUM boundary: {low_medium_threshold:.0f}")
    print(f"  MEDIUM/HIGH boundary: {medium_high_threshold:.0f}")

    return {
        'low_medium': low_medium_threshold,
        'medium_high': medium_high_threshold
    }
```

### 13.3 Visualization of Thresholds

```
┌─────────────────────────────────────────────────────────────────────┐
│    FRAUD SCORE DISTRIBUTION BY CLASS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    Legitimate Farms (450)      │        Fraud Farms (50)            │
│    █████████████████████       │   ░░░░░░░░░░░░░░░░░░░░░           │
│    █████████████████████████   │       ░░░░░░░░░░░░░░░░░░░░░░░░░   │
│    ███████████████████████████ │           ░░░░░░░░░░░░░░░░░░░░░   │
│    ├─────────────┼─────────────┼─────────────┼─────────────┼────   │
│    0            35            65            85          100         │
│                  ↑             ↑                                    │
│            LOW/MEDIUM    MEDIUM/HIGH                                │
│                                                                      │
│    GREEN ZONE    │  YELLOW ZONE  │         RED ZONE                │
│    (APPROVE)     │  (REVIEW)     │         (REJECT)                │
└─────────────────────────────────────────────────────────────────────┘
```

### 13.4 Balancing False Positives vs False Negatives

```
BUSINESS DECISION: What's worse?

FALSE POSITIVE (Flagging legitimate farmer as fraud):
├── Farmer is wrongly denied insurance/credit
├── Reputation damage to system
├── Legal liability
└── Cost: Loss of customer, potential lawsuit

FALSE NEGATIVE (Missing actual fraud):
├── Fraudulent claim is paid
├── Financial loss
├── Encourages more fraud
└── Cost: Direct financial loss

TYPICAL APPROACH:
├── For insurance: FP rate < 10% is critical (don't reject good farmers)
├── For subsidies: FN rate < 5% is critical (don't waste public funds)

ADJUST THRESHOLDS ACCORDINGLY:
├── If FP too high: RAISE the LOW/MEDIUM threshold
├── If FN too high: LOWER the MEDIUM/HIGH threshold
```

---

## 14. VALIDATION & TESTING PROTOCOL

### 14.1 Validation Metrics

| Metric                  | Formula              | Target | Meaning                                   |
| ----------------------- | -------------------- | ------ | ----------------------------------------- |
| **Precision**           | TP / (TP + FP)       | > 90%  | Of flagged farms, how many are real fraud |
| **Recall**              | TP / (TP + FN)       | > 85%  | Of real fraud, how many did we catch      |
| **F1 Score**            | 2 × P × R / (P + R)  | > 87%  | Harmonic mean of precision and recall     |
| **False Positive Rate** | FP / (FP + TN)       | < 10%  | Legitimate farms wrongly flagged          |
| **AUC-ROC**             | Area under ROC curve | > 0.90 | Overall discrimination ability            |

### 14.2 Validation Procedure

```python
def validate_calibration(self):
    """
    Validate the calibrated scoring system on test data.
    """
    from sklearn.metrics import classification_report, confusion_matrix, roc_auc_score

    # Calculate fraud score for test set using calibrated thresholds
    test_scores = self.calculate_fraud_scores(self.test_df)

    # Apply risk thresholds
    predictions = np.where(test_scores >= self.medium_high_threshold, 1, 0)

    # Get actual fraud labels
    actual = self.test_df['is_fraud']

    # Calculate metrics
    print("Classification Report:")
    print(classification_report(actual, predictions))

    print("\nConfusion Matrix:")
    cm = confusion_matrix(actual, predictions)
    print(f"  TN={cm[0,0]}, FP={cm[0,1]}")
    print(f"  FN={cm[1,0]}, TP={cm[1,1]}")

    # Calculate rates
    tn, fp, fn, tp = cm.ravel()
    fpr = fp / (fp + tn)
    fnr = fn / (fn + tp)

    print(f"\nFalse Positive Rate: {fpr:.1%}")
    print(f"False Negative Rate: {fnr:.1%}")
    print(f"AUC-ROC: {roc_auc_score(actual, test_scores):.3f}")
```

### 14.3 Cross-Validation

```python
def cross_validate(self, n_folds=5):
    """
    Perform k-fold cross-validation to ensure robustness.
    """
    from sklearn.model_selection import StratifiedKFold

    kfold = StratifiedKFold(n_splits=n_folds, shuffle=True, random_state=42)

    fold_results = []
    for fold, (train_idx, val_idx) in enumerate(kfold.split(self.df, self.df['is_fraud'])):
        # Train on fold
        train_data = self.df.iloc[train_idx]
        val_data = self.df.iloc[val_idx]

        # Calibrate on train
        thresholds = self.calibrate_all(train_data)

        # Validate on val
        val_scores = self.calculate_fraud_scores(val_data, thresholds)
        auc = roc_auc_score(val_data['is_fraud'], val_scores)

        fold_results.append(auc)
        print(f"Fold {fold+1}: AUC = {auc:.3f}")

    print(f"\nMean AUC: {np.mean(fold_results):.3f} ± {np.std(fold_results):.3f}")
```

---

## 15. CONTINUOUS IMPROVEMENT PROCESS

### 15.1 Monitoring in Production

```
MONTHLY MONITORING CHECKLIST:

□ Calculate actual fraud rate from verified cases
□ Compare predicted fraud rate to actual
□ Review false positive complaints
□ Review missed fraud cases (false negatives)
□ Check indicator score distributions
□ Identify any drift in data patterns
```

### 15.2 When to Recalibrate

```
RECALIBRATION TRIGGERS:

1. PERFORMANCE DEGRADATION
   ├── F1 score drops below 85%
   ├── False positive rate exceeds 15%
   └── False negative rate exceeds 10%

2. DATA DRIFT
   ├── New crop types added
   ├── Geographic expansion to new regions
   └── Seasonal patterns change

3. EXTERNAL CHANGES
   ├── New satellite data sources
   ├── Changes in fraud patterns
   └── Policy or regulatory changes

4. SCHEDULED REVIEW
   └── At minimum, annually
```

### 15.3 Recalibration Process

```
RECALIBRATION WORKFLOW:

1. COLLECT NEW LABELED DATA
   ├── Include recent fraud cases
   ├── Include verified legitimate farms
   └── Ensure geographic and crop diversity

2. MERGE WITH HISTORICAL DATA
   ├── Weight recent data more heavily
   └── Or use only last 2 years of data

3. RUN CALIBRATION PIPELINE
   ├── Recalculate thresholds
   ├── Recalculate weights
   └── Recalibrate risk levels

4. VALIDATE NEW CALIBRATION
   ├── Compare to previous calibration
   ├── Ensure no regression
   └── Test on held-out data

5. DEPLOY NEW CALIBRATION
   ├── A/B test if possible
   └── Monitor closely for first week
```

---

## APPENDIX: COMPLETE CALIBRATION CODE

### Full Python Implementation

```python
#!/usr/bin/env python3
"""
Meridian Sentinel - Fraud Scoring Calibration System
Complete implementation for calibrating all 7 indicators.
"""

import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split, StratifiedKFold
from sklearn.metrics import roc_curve, auc, classification_report, confusion_matrix
from sklearn.linear_model import LogisticRegression

class MeridianSentinelCalibrator:
    """Complete calibration system for fraud detection scoring."""

    def __init__(self, labeled_data_path):
        self.df = pd.read_csv(labeled_data_path)
        self.calibrated_thresholds = {}
        self.calibrated_weights = {}
        self.risk_thresholds = {}

    def run_full_calibration(self):
        """Execute complete calibration pipeline."""
        print("=" * 60)
        print("MERIDIAN SENTINEL CALIBRATION SYSTEM")
        print("=" * 60)

        # Step 1: Prepare data
        self.prepare_data()

        # Step 2: Calibrate each indicator
        self.calibrate_size_discrepancy()
        self.calibrate_crop_mismatch()
        self.calibrate_weather_validation()
        self.calibrate_ghost_farmer()
        self.calibrate_historical_consistency()
        self.calibrate_disaster_validation()
        self.calibrate_cropland_signal()

        # Step 3: Calculate optimal weights
        self.calculate_optimal_weights()

        # Step 4: Calibrate risk thresholds
        self.calibrate_risk_thresholds()

        # Step 5: Validate
        self.validate_calibration()

        # Step 6: Export results
        self.export_calibration()

        print("=" * 60)
        print("CALIBRATION COMPLETE")
        print("=" * 60)

    def export_calibration(self):
        """Export calibrated values to JSON."""
        import json

        calibration = {
            'thresholds': self.calibrated_thresholds,
            'weights': self.calibrated_weights,
            'risk_levels': self.risk_thresholds,
            'calibration_date': str(pd.Timestamp.now()),
            'dataset_size': len(self.df),
            'fraud_rate': float(self.df['is_fraud'].mean())
        }

        with open('calibration_output.json', 'w') as f:
            json.dump(calibration, f, indent=2)

        print("\nCalibration exported to calibration_output.json")

# Usage:
# calibrator = MeridianSentinelCalibrator('labeled_farms.csv')
# calibrator.run_full_calibration()
```

---

## SUMMARY: COMPLETE SCORING PROCESS

### Step-by-Step How Points Are Calculated

```
STEP 1: COLLECT FARMER APPLICATION DATA
├── Claimed farm area (hectares)
├── Claimed crop type
├── Planting date
├── Farm location (coordinates)
├── Disaster claim (if any)

STEP 2: EXTRACT SATELLITE DATA FROM GEE
├── Detected farm boundary & area
├── NDVI values (current and historical)
├── Rainfall data (6-month period)
├── Population density (WorldPop)
├── Land cover classification (Dynamic World)

STEP 3: CALCULATE EACH INDICATOR

   INDICATOR 1 - SIZE DISCREPANCY:
   │ Difference = |Detected - Claimed| / Claimed × 100
   │ IF difference <= 15%: score = 0
   │ ELSE IF <= 30%: score = 5
   │ ELSE IF <= 50%: score = 15
   │ ELSE IF <= 70%: score = 25
   │ ELSE: score = 30

   INDICATOR 2 - CROP MISMATCH:
   │ IF crops match: score = 0
   │ ELSE IF confidence > 80%: score = 30
   │ ELSE IF confidence > 65%: score = 20
   │ ELSE: score = 10

   INDICATOR 3 - WEATHER VALIDATION:
   │ ratio = actual_rainfall / required_rainfall
   │ IF ratio < 0.50: score = 20
   │ ELSE IF ratio < 0.70: score = 15
   │ ELSE IF ratio < 0.85: score = 8
   │ ELSE: score = 0

   INDICATOR 4 - GHOST FARMER:
   │ density = population per hectare
   │ IF density < 1: score = 20
   │ ELSE IF density < 5: score = 10
   │ ELSE: score = 0

   INDICATOR 5 - HISTORICAL CONSISTENCY:
   │ change = |current_NDVI - historical_NDVI|
   │ IF change > 0.4: score = 15
   │ ELSE IF change > 0.2: score = 10
   │ ELSE: score = 0
   │ IF forest_to_cropland: score += 15

   INDICATOR 6 - DISASTER VALIDATION:
   │ IF no_disaster_claim: score = 0
   │ ELSE IF disaster_confirmed: score = 0
   │ ELSE: score = 10

   INDICATOR 7 - CROPLAND SIGNAL:
   │ IF cropland_prob < 30% AND NDVI < 0.35: score = 10
   │ ELSE: score = 0

STEP 4: CALCULATE TOTAL FRAUD SCORE
   total_raw = sum(all_indicator_scores)  // Max: 150
   scaled_score = (total_raw / 150) × 100  // Range: 0-100

STEP 5: ASSIGN RISK LEVEL
   IF scaled_score >= 70: risk = "HIGH" → REJECT
   ELSE IF scaled_score >= 40: risk = "MEDIUM" → MANUAL_REVIEW
   ELSE: risk = "LOW" → APPROVE
```

### Example Calculation

```
EXAMPLE FARM APPLICATION:

INPUT:
├── Claimed area: 2.0 hectares
├── Claimed crop: Maize
├── Location: Kenya (0.5°N, 37.2°E)
├── Planting date: March 2024
├── Disaster claim: None

SATELLITE DATA:
├── Detected area: 1.3 hectares
├── Detected crop: Maize (85% confidence)
├── 6-month rainfall: 380mm (vs 450mm required)
├── Population density: 12 people/hectare
├── Historical NDVI change: 0.15
├── Cropland probability: 72%
├── Current NDVI: 0.52

SCORING:
├── Size: |1.3-2.0|/2.0 = 35% → 15 points
├── Crop: Match with high conf → 0 points
├── Weather: 380/450 = 84% → 8 points
├── Ghost: 12 people/ha → 0 points
├── Historical: 0.15 change → 0 points
├── Disaster: No claim → 0 points
├── Cropland: 72% prob, 0.52 NDVI → 0 points

TOTAL: 15 + 0 + 8 + 0 + 0 + 0 + 0 = 23 points
SCALED: (23/150) × 100 = 15.3

RESULT: LOW RISK → AUTO-APPROVE
```

### Official Weight Summary

| Indicator              | Max Points | Percentage |
| ---------------------- | ---------- | ---------- |
| Size Discrepancy       | 30         | 20.0%      |
| Crop Type Mismatch     | 30         | 20.0%      |
| Weather Validation     | 20         | 13.3%      |
| Ghost Farmer Detection | 20         | 13.3%      |
| Historical Consistency | 15 (+15)   | 10.0%      |
| Disaster Claim Valid.  | 10         | 6.7%       |
| Cropland Signal        | 10         | 6.7%       |
| **TOTAL**              | **150**    | **100%**   |

---

_Document End - Scoring Calibration Methodology_
_Version 1.0 | December 2024_
_Meridian Sentinel - Agricultural Fraud Detection System_



TOTAL POSSIBLE: 150 points → Scaled to 0-100

INDICATOR 1: SIZE DISCREPANCY (30 pts)
├── 0-15%:  0  │  15-30%:  5  │  30-50%: 15  │  50-70%: 25  │  >70%: 30

INDICATOR 2: CROP MISMATCH (30 pts)
├── Match: 0  │  High conf: 30  │  Med conf: 20  │  Low conf: 10

INDICATOR 3: WEATHER (20 pts)
├── <50%: 20  │  50-70%: 15  │  70-85%: 8  │  >85%: 0

INDICATOR 4: GHOST FARMER (20 pts)
├── <1/ha: 20  │  1-5/ha: 10  │  >5/ha: 0

INDICATOR 5: HISTORICAL (15 pts + 15 bonus)
├── >0.4: 15  │  0.2-0.4: 10  │  <0.2: 0  │  Forest→Crop: +15

INDICATOR 6: DISASTER (10 pts)
├── No claim: 0  │  Confirmed: 0  │  No evidence: 10

INDICATOR 7: CROPLAND (10 pts)
├── Prob <30% AND NDVI <0.35: 10  │  Otherwise: 0

RISK LEVELS:
├── 0-39:   LOW    → AUTO-APPROVE
├── 40-69:  MEDIUM → MANUAL REVIEW
└── 70-100: HIGH   → REJECT