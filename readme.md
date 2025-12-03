# MERIDIAN SENTINEL - Project Analysis & Requirements Documentation

## Document Version: 1.0

## Date: December 3, 2024

## Status: Final Analysis

---

## TABLE OF CONTENTS

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement Analysis](#2-problem-statement-analysis)
3. [Solution Overview](#3-solution-overview)
4. [Core Requirements Analysis](#4-core-requirements-analysis)
5. [Fraud Detection Requirements](#5-fraud-detection-requirements)
6. [Use Case Analysis](#6-use-case-analysis)
7. [Deliverables Matrix](#7-deliverables-matrix)
8. [Success Criteria](#8-success-criteria)
9. [Data Sources Requirements](#9-data-sources-requirements)
10. [Risk Assessment](#10-risk-assessment)

---

## 1. EXECUTIVE SUMMARY

### What is Meridian Sentinel?

Meridian Sentinel is an **automated farm verification and fraud detection system** designed specifically for:

- Agricultural Insurance Companies
- Government Subsidy Programs
- Agricultural Credit/Microfinance Institutions
- Operating primarily in Sub-Saharan Africa

### Core Value Proposition

| Current State                              | With Meridian Sentinel              |
| ------------------------------------------ | ----------------------------------- |
| Manual verification: 2-3 weeks/application | Automated: <10 seconds/farm         |
| Fraud rate: 15-20%                         | Fraud detection: 90%+ accuracy      |
| Expensive field visits                     | Remote satellite-based verification |
| Subjective assessments                     | Objective, data-driven decisions    |

### Technology Stack

**Single Platform Approach**: Google Earth Engine (GEE) exclusively for:

- Satellite imagery processing
- Geospatial analysis
- Fraud indicator calculation
- Real-time risk scoring

---

## 2. PROBLEM STATEMENT ANALYSIS

### 2.1 Current Challenges

| Challenge                    | Impact                          | Scale                             |
| ---------------------------- | ------------------------------- | --------------------------------- |
| **Slow Manual Verification** | 2-3 weeks per application       | Millions of applications annually |
| **High Fraud Rate**          | 15-20% of claims are fraudulent | Significant financial losses      |
| **Expensive Operations**     | Field visits, manual labor      | Unsustainable at scale            |
| **Inconsistent Assessments** | Subjective human judgment       | Quality variation                 |
| **Limited Coverage**         | Cannot verify all applications  | Only spot-checks (5-10%)          |

### 2.2 Fraud Types Identified

1. **Size Inflation** - Farmers claim larger areas than actual
2. **Crop Misrepresentation** - Claiming high-value crops while growing low-value ones
3. **Ghost Farmers** - Fake registrations in uninhabited areas
4. **False Disaster Claims** - Fabricated flood/drought damage
5. **Land Conversion Fraud** - Recently cleared forest claimed as established farmland
6. **Non-Agricultural Land** - Claims on urban, forest, or water bodies

### 2.3 Stakeholder Analysis

| Stakeholder               | Primary Need                        | Key Metric                            |
| ------------------------- | ----------------------------------- | ------------------------------------- |
| Insurance Companies       | Reduce fraud, speed up claims       | Processing time, fraud detection rate |
| Government Programs       | Ensure subsidies reach real farmers | Ghost farmer detection, coverage      |
| Microfinance Institutions | Assess farm viability for loans     | Health scoring, risk assessment       |
| Farmers (Legitimate)      | Fast, fair approval process         | False positive rate <10%              |

---

## 3. SOLUTION OVERVIEW

### 3.1 System Capabilities Required

```
┌─────────────────────────────────────────────────────────────────┐
│                    MERIDIAN SENTINEL CORE                        │
├─────────────────────────────────────────────────────────────────┤
│  INPUT: GPS Coordinates (lat/long) + Farmer Claims               │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  FIELD      │  │   CROP      │  │   CROP      │              │
│  │  BOUNDARY   │  │   TYPE      │  │   HEALTH    │              │
│  │  DETECTION  │  │   CLASSIF.  │  │   MONITOR   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   AREA      │  │   WEATHER   │  │  HISTORICAL │              │
│  │   CALC.     │  │   VALID.    │  │   ANALYSIS  │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                   │
├─────────────────────────────────────────────────────────────────┤
│              FRAUD DETECTION ENGINE (8 Indicators)               │
├─────────────────────────────────────────────────────────────────┤
│  OUTPUT: Risk Score (0-100) + Recommendation + Evidence          │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Processing Requirements

| Requirement              | Target               | Priority |
| ------------------------ | -------------------- | -------- |
| Processing Speed         | <10 seconds per farm | CRITICAL |
| Batch Capacity           | 100,000+ farms       | HIGH     |
| Fraud Detection Accuracy | >85%                 | CRITICAL |
| False Positive Rate      | <10%                 | CRITICAL |
| Boundary Detection       | 95%+ success rate    | HIGH     |
| Crop Classification      | 90%+ accuracy        | HIGH     |

---

## 4. CORE REQUIREMENTS ANALYSIS

### 4.1 Field Boundary Detection

**Objective**: Automatically identify farm boundaries from GPS coordinates

| Requirement      | Specification                | Validation Method |
| ---------------- | ---------------------------- | ----------------- |
| Input            | GPS coordinates (lat/long)   | API validation    |
| Detection Radius | 300m from point              | Configuration     |
| Farm Size Range  | 0.1-10 hectares              | Edge case testing |
| Accuracy Target  | 75%+ boundary detection      | IoU scoring       |
| Mixed Land Use   | Must handle fragmented farms | Field testing     |

**Technical Approach**:

- **Primary**: SNIC (Simple Non-Iterative Clustering) on Sentinel-2
- **Fallback**: Dynamic World pre-classified cropland layer
- **Validation**: NDVI thresholding to confirm cropland

**Success Metrics**:

- Boundary detected for 95%+ of valid farms
- Area calculation within ±20% of ground truth
- Processing time: <8 seconds per farm

### 4.2 Crop Classification

**Objective**: Identify crop type from satellite spectral signatures

| Crop Type | Priority | Technical Feasibility       |
| --------- | -------- | --------------------------- |
| Maize     | HIGH     | Well-documented signatures  |
| Rice      | HIGH     | Distinct water requirements |
| Cassava   | HIGH     | Unique growth patterns      |
| Sorghum   | MEDIUM   | Similar to maize            |
| Beans     | MEDIUM   | Legume signatures           |
| Millet    | MEDIUM   | Distinct spectral profile   |

**Technical Approach**:

- Rules-based classification using NDVI/EVI thresholds
- Multi-temporal analysis for phenological patterns
- Crop-specific spectral signatures from literature
- Optional: Random Forest if training data available

**Success Metrics**:

- 90%+ classification accuracy
- Confidence scores correlate with accuracy
- Balanced performance across all crop types

### 4.3 Crop Health Monitoring

**Objective**: Assess current crop health and stress levels

| Health Status | NDVI Range | Interpretation      |
| ------------- | ---------- | ------------------- |
| Healthy       | >0.6       | Optimal growth      |
| Stressed      | 0.4-0.6    | Intervention needed |
| Critical      | 0.2-0.4    | Significant damage  |
| Failed        | <0.2       | Crop loss           |

**Stress Detection Types**:

1. **Water Stress** - Via Sentinel-1 SAR soil moisture
2. **Nutrient Deficiency** - Via spectral analysis
3. **General Stress** - Via NDVI anomaly detection

**Success Metrics**:

- Health status matches ground truth 80%+ of time
- Early stress detection (2-4 weeks before visible)
- False alarm rate <10%

### 4.4 Area Calculation & Validation

**Objective**: Calculate accurate farm area and detect size fraud

| Method               | Description             | Accuracy   |
| -------------------- | ----------------------- | ---------- |
| Geodesic Calculation | True earth surface area | Primary    |
| Pixel Counting       | Grid-based validation   | Secondary  |
| Shape Complexity     | Perimeter/area ratio    | Validation |

**Fraud Detection Thresholds**:

- 0-15% difference: Normal variation (acceptable)
- 15-30% difference: Minor discrepancy (flag)
- 30-50% difference: Moderate discrepancy (review)
- > 50% difference: Major discrepancy (reject)

**Success Metrics**:

- Area within ±15% of ground truth 90%+ of time
- Size fraud detection rate >90%
- False positive rate <5%

---

## 5. FRAUD DETECTION REQUIREMENTS

### 5.1 Eight-Indicator Scoring System

| #   | Indicator                 | Weight | Max Points | Data Source              |
| --- | ------------------------- | ------ | ---------- | ------------------------ |
| 1   | Size Discrepancy          | 30     | 30         | Sentinel-2 + Calculation |
| 2   | Crop Type Mismatch        | 30     | 30         | Sentinel-2 Spectral      |
| 3   | Weather Validation        | 20     | 20         | CHIRPS Rainfall          |
| 4   | Ghost Farmer Detection    | 20     | 20         | WorldPop                 |
| 5   | Historical Consistency    | 15     | 15         | Multi-temporal NDVI      |
| 6   | Disaster Claim Validation | 10     | 10         | Sentinel-1 + GSW         |
| 7   | Cropland Signal           | 10     | 10         | Dynamic World + NDVI     |
| 8   | Infrastructure Presence   | -      | -          | Google Open Buildings    |

**Note**: Document mentions 8 indicators but details 7. Infrastructure presence (mentioned in requirements) appears to be the 8th.

### 5.2 Risk Classification

| Score Range | Risk Level      | Action             | Expected % |
| ----------- | --------------- | ------------------ | ---------- |
| 0-39        | LOW (Green)     | Auto-approve       | 80-85%     |
| 40-69       | MEDIUM (Yellow) | Manual review      | 10-15%     |
| 70-100      | HIGH (Red)      | Reject/Investigate | 5-10%      |

### 5.3 Evidence Package Requirements

Each verification must include:

- [ ] Fraud score (0-100)
- [ ] Risk level (Low/Medium/High)
- [ ] Recommendation (Approve/Review/Reject)
- [ ] Individual indicator scores
- [ ] Supporting evidence (maps, imagery, charts)
- [ ] Confidence scores for each check
- [ ] Human-readable explanation

---

## 6. USE CASE ANALYSIS

### 6.1 Use Case Matrix

| Use Case               | Volume       | Speed Requirement | Key Metrics            |
| ---------------------- | ------------ | ----------------- | ---------------------- |
| Agricultural Insurance | 500,000 apps | 3-4 days batch    | Fraud detection rate   |
| Government Subsidies   | 1M+ farmers  | Regional batches  | Ghost farmer detection |
| Agricultural Credit    | Real-time    | <10 seconds       | Health scoring         |
| Continuous Monitoring  | 10,000 farms | Weekly updates    | Anomaly detection      |

### 6.2 Use Case 1: Agricultural Insurance Verification

**Scenario**: Insurance company receives 500,000 crop insurance applications

**Current vs. Meridian**:
| Metric | Current | With Meridian |
|--------|---------|---------------|
| Verification time | 1-2 days/app | 8-10 seconds/app |
| Total processing | 500-1000 days | 3-4 days |
| Fraud detection | 15-20% fraud rate | 90%+ detection |
| Cost reduction | Baseline | 98% reduction |

**Requirements**:

- Batch process 500K farms
- Export results to CSV
- Generate risk distribution report
- Flag high-risk (5-10%) for manual review
- Provide evidence packages for rejections

### 6.3 Use Case 2: Government Subsidy Program

**Scenario**: Government provides fertilizer subsidies to 1M+ smallholder farmers

**Key Fraud Risks**:

1. Ghost farmers (fake registrations)
2. Size inflation (larger claims)
3. Duplicate registrations
4. Non-existent farms

**Requirements**:

- Verify 100% of applications (not just 5-10% spot checks)
- Regional processing (divide country into zones)
- Multi-crop support
- Offline report generation
- Appeals process for flagged applications

**Impact**:

- Eliminate ghost farmers: Save 10-15% of budget
- Reduce size fraud: Save 15-20% of budget
- Ensure subsidies reach real farmers

### 6.4 Use Case 3: Agricultural Credit Assessment

**Scenario**: Microfinance institution evaluates farm-backed loans

**Credit Scoring Integration**:

```
Farm Viability Score = {
    Verified farm size (objective measurement)
    + Crop health score (0-100)
    + Historical consistency (stable farmland)
    + Weather conditions (adequate rainfall)
    + Infrastructure presence (established operation)
}
```

**Loan Decision Matrix**:
| Farm Viability | Fraud Score | Decision |
|----------------|-------------|----------|
| High | Low | Approve |
| Medium | Low | Approve with monitoring |
| Any | High | Reject |
| Low | Any | Reject |

**Requirements**:

- Real-time API for instant decisions
- Health monitoring dashboard
- Season-long tracking
- Risk alerts if crops deteriorate

### 6.5 Use Case 4: Continuous Crop Monitoring

**Scenario**: Monitor 10,000 insured farms throughout growing season

**Monitoring Capabilities**:

- Weekly automated monitoring
- Early stress detection (2-4 weeks advance warning)
- Automated alerts for farmer intervention
- Evidence-based claim validation

**Requirements**:

- Time-series analysis for each farm
- Anomaly detection algorithms
- Historical comparison
- Export capabilities for reporting

---

## 7. DELIVERABLES MATRIX

### 7.1 Module Deliverables

| Deliverable                      | Components                                                                      | Acceptance Criteria              |
| -------------------------------- | ------------------------------------------------------------------------------- | -------------------------------- |
| **1.1 Field Boundary Detection** | SNIC segmentation, Dynamic World fallback, Area calculation, Confidence scoring | 95%+ accuracy on 50 test farms   |
| **1.2 Crop Classification**      | Rules-based classification, NDVI/EVI thresholds, Confidence calculation         | 90%+ accuracy on 100 test farms  |
| **1.3 Health Monitoring**        | Health assessment, NDVI/EVI/SAVI indices, Stress detection, Health score        | 90%+ correlation with field data |
| **1.4 Fraud Detection System**   | All 7 indicators, Scoring algorithm, Risk classification, Evidence packages     | F1 score >90%, FPR <10%          |
| **1.5 Weather Validation**       | CHIRPS integration, Crop water requirements, Drought detection                  | Accurate within 10% of actuals   |
| **1.6 Historical Analysis**      | Multi-temporal change detection, Deforestation detection, Land use scoring      | Change detection >85% accuracy   |

### 7.2 Testing Requirements

| Module              | Test Farms | Accuracy Target  | Processing Time |
| ------------------- | ---------- | ---------------- | --------------- |
| Boundary Detection  | 50 farms   | 95%+             | <8 seconds      |
| Crop Classification | 100 farms  | 90%+             | <5 seconds      |
| Health Monitoring   | 50 farms   | 90%+ correlation | <5 seconds      |
| Complete Pipeline   | 200 farms  | F1 >90%          | <12 seconds     |

---

## 8. SUCCESS CRITERIA

### 8.1 Technical Performance Metrics

| Metric                       | Target | Measurement Method                    | Priority |
| ---------------------------- | ------ | ------------------------------------- | -------- |
| Boundary Detection Accuracy  | >92%   | IoU score on 100 test farms           | HIGH     |
| Crop Classification Accuracy | >90%   | Confusion matrix on 200 test farms    | HIGH     |
| Fraud Detection F1 Score     | >90%   | Validated against known fraud cases   | CRITICAL |
| False Positive Rate          | <10%   | Legitimate farms incorrectly flagged  | CRITICAL |
| Error Rate                   | <2%    | Failed verifications / total attempts | HIGH     |
| Processing Time              | <12s   | End-to-end per farm                   | HIGH     |

### 8.2 Business Impact Metrics

| Metric                      | Target            | Baseline                  |
| --------------------------- | ----------------- | ------------------------- |
| Processing Time Reduction   | 99%+              | 2-3 weeks → seconds       |
| Cost Reduction              | 98%+              | Manual verification costs |
| Fraud Detection Improvement | 85%+ catch rate   | 15-20% fraud rate         |
| Coverage Improvement        | 100% verification | 5-10% spot checks         |

---

## 9. DATA SOURCES REQUIREMENTS

### 9.1 Primary Data Sources (All via GEE)

| Data Source           | Purpose                                    | Resolution     | Update Frequency |
| --------------------- | ------------------------------------------ | -------------- | ---------------- |
| Sentinel-2            | Optical imagery, NDVI, crop classification | 10m            | 5 days           |
| Sentinel-1            | SAR for flood detection, soil moisture     | 10m            | 6 days           |
| Dynamic World         | Pre-classified land cover                  | 10m            | Near real-time   |
| CHIRPS                | Rainfall data                              | 5km            | Daily            |
| WorldPop              | Population density                         | 100m           | Annual           |
| Google Open Buildings | Infrastructure detection                   | Building-level | Periodic         |
| SRTM                  | Elevation (optional)                       | 30m            | Static           |
| Global Surface Water  | Flood-prone areas                          | 30m            | Monthly          |

### 9.2 Data Quality Requirements

| Requirement       | Specification                   |
| ----------------- | ------------------------------- |
| Cloud cover       | <20% for valid imagery          |
| Temporal coverage | 6-month historical minimum      |
| Spatial coverage  | All Sub-Saharan Africa          |
| Data freshness    | <30 days for current assessment |

---

## 10. RISK ASSESSMENT

### 10.1 Technical Risks

| Risk                                | Impact | Likelihood | Mitigation                                 |
| ----------------------------------- | ------ | ---------- | ------------------------------------------ |
| Cloud cover limiting imagery        | HIGH   | MEDIUM     | Multi-date compositing, SAR fallback       |
| Processing timeout on large batches | MEDIUM | MEDIUM     | Chunked processing, async operations       |
| GEE quota limits                    | HIGH   | LOW        | Batch optimization, caching                |
| Spectral confusion between crops    | MEDIUM | MEDIUM     | Multi-temporal analysis, confidence scores |

### 10.2 Business Risks

| Risk                                        | Impact | Likelihood | Mitigation                             |
| ------------------------------------------- | ------ | ---------- | -------------------------------------- |
| False positives blocking legitimate farmers | HIGH   | MEDIUM     | <10% FPR target, appeals process       |
| Fraudsters gaming the system                | MEDIUM | MEDIUM     | Multiple indicators, anomaly detection |
| Regulatory compliance                       | HIGH   | LOW        | Evidence packages, audit trails        |

### 10.3 Operational Risks

| Risk                         | Impact | Likelihood | Mitigation                        |
| ---------------------------- | ------ | ---------- | --------------------------------- |
| GEE service outage           | HIGH   | LOW        | Caching, offline fallback         |
| Internet connectivity issues | MEDIUM | MEDIUM     | Batch processing, offline reports |
| Staff training               | MEDIUM | MEDIUM     | Documentation, training materials |

---

## APPENDIX A: GLOSSARY

| Term         | Definition                                                           |
| ------------ | -------------------------------------------------------------------- |
| **NDVI**     | Normalized Difference Vegetation Index - vegetation health indicator |
| **EVI**      | Enhanced Vegetation Index - improved vegetation index                |
| **SAVI**     | Soil Adjusted Vegetation Index                                       |
| **SAR**      | Synthetic Aperture Radar - works through clouds                      |
| **SNIC**     | Simple Non-Iterative Clustering - image segmentation algorithm       |
| **IoU**      | Intersection over Union - boundary detection accuracy metric         |
| **F1 Score** | Harmonic mean of precision and recall                                |
| **GEE**      | Google Earth Engine                                                  |
| **CHIRPS**   | Climate Hazards Group InfraRed Precipitation with Station data       |

---

## APPENDIX B: REQUIREMENT TRACEABILITY

| Requirement ID | Description                | Module   | Status       |
| -------------- | -------------------------- | -------- | ------------ |
| REQ-001        | Field boundary detection   | 1.1      | To Implement |
| REQ-002        | Crop type classification   | 1.2      | To Implement |
| REQ-003        | Crop health assessment     | 1.3      | To Implement |
| REQ-004        | Area calculation           | 1.1, 1.4 | To Implement |
| REQ-005        | Size discrepancy detection | 1.4      | To Implement |
| REQ-006        | Crop mismatch detection    | 1.4      | To Implement |
| REQ-007        | Weather validation         | 1.5      | To Implement |
| REQ-008        | Ghost farmer detection     | 1.4      | To Implement |
| REQ-009        | Historical consistency     | 1.6      | To Implement |
| REQ-010        | Disaster validation        | 1.4      | To Implement |
| REQ-011        | Cropland signal validation | 1.4      | To Implement |
| REQ-012        | Fraud scoring              | 1.4      | To Implement |
| REQ-013        | Risk classification        | 1.4      | To Implement |
| REQ-014        | Evidence packages          | 1.4      | To Implement |
| REQ-015        | Batch processing           | System   | To Implement |
| REQ-016        | API access                 | System   | To Implement |

---

_Document End - Project Analysis & Requirements_
_Version 1.0 | December 2024_
