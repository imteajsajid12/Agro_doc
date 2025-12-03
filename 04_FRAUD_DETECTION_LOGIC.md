# FRAUD DETECTION LOGIC DOCUMENTATION

## Meridian Sentinel - Agricultural Fraud Detection System

**Document Version:** 1.0  
**Date:** December 2024  
**Classification:** Technical Implementation Guide

---

## TABLE OF CONTENTS

1. [Executive Summary](#1-executive-summary)
2. [Fraud Detection Overview](#2-fraud-detection-overview)
3. [Indicator 1: Size Discrepancy](#3-indicator-1-size-discrepancy)
4. [Indicator 2: Crop Type Mismatch](#4-indicator-2-crop-type-mismatch)
5. [Indicator 3: Weather Validation](#5-indicator-3-weather-validation)
6. [Indicator 4: Ghost Farmer Detection](#6-indicator-4-ghost-farmer-detection)
7. [Indicator 5: Historical Consistency](#7-indicator-5-historical-consistency)
8. [Indicator 6: Disaster Claim Validation](#8-indicator-6-disaster-claim-validation)
9. [Indicator 7: Cropland Signal](#9-indicator-7-cropland-signal)
10. [Scoring Algorithm](#10-scoring-algorithm)
11. [Risk Classification](#11-risk-classification)
12. [Evidence Package Generation](#12-evidence-package-generation)
13. [End-to-End Workflow Example](#13-end-to-end-workflow-example)
14. [GEE Feasibility Summary](#14-gee-feasibility-summary)
15. [Validation & Testing](#15-validation--testing)

---

## 1. EXECUTIVE SUMMARY

### 1.1 Purpose

This document provides a **complete, step-by-step technical guide** for implementing the fraud detection logic in the Meridian Sentinel system. It covers:

- **What each indicator detects** and why it matters
- **How to calculate** each indicator using Google Earth Engine
- **Thresholds and scoring** for each indicator
- **Complete GEE code** for implementation
- **Feasibility analysis** for each component

### 1.2 Fraud Detection Philosophy

The system uses a **multi-indicator weighted scoring approach**:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FRAUD DETECTION PHILOSOPHY                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PRINCIPLE 1: No Single Point of Failure                            │
│  ─────────────────────────────────────────                          │
│  No single indicator can flag a farm as fraudulent.                 │
│  Multiple indicators must align to raise the fraud score.           │
│                                                                      │
│  PRINCIPLE 2: Weighted by Reliability                               │
│  ─────────────────────────────────────────                          │
│  Indicators with higher accuracy get higher weights.                │
│  Size discrepancy (30 pts) > Cropland signal (10 pts)              │
│                                                                      │
│  PRINCIPLE 3: Evidence-Based                                        │
│  ─────────────────────────────────────────                          │
│  Every flag must be accompanied by visual/data evidence.            │
│  Human reviewers can verify the system's findings.                  │
│                                                                      │
│  PRINCIPLE 4: Conservative Thresholds                               │
│  ─────────────────────────────────────────                          │
│  Thresholds are set to minimize false positives.                    │
│  Better to miss some fraud than wrongly accuse farmers.             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.3 Indicator Summary Table

| #         | Indicator                 | Max Points | Weight   | GEE Available | Reliability |
| --------- | ------------------------- | ---------- | -------- | ------------- | ----------- |
| 1         | Size Discrepancy          | 30         | 22.2%    | ✅ Yes        | HIGH        |
| 2         | Crop Type Mismatch        | 30         | 22.2%    | ✅ Yes        | HIGH        |
| 3         | Weather Validation        | 20         | 14.8%    | ✅ Yes        | HIGH        |
| 4         | Ghost Farmer Detection    | 20         | 14.8%    | ✅ Yes        | MEDIUM      |
| 5         | Historical Consistency    | 15         | 11.1%    | ✅ Yes        | MEDIUM      |
| 6         | Disaster Claim Validation | 10         | 7.4%     | ✅ Yes        | HIGH        |
| 7         | Cropland Signal           | 10         | 7.4%     | ✅ Yes        | HIGH        |
| **TOTAL** |                           | **135**    | **100%** |               |             |

---

## 2. FRAUD DETECTION OVERVIEW

### 2.1 How Fraud Detection Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FRAUD DETECTION FLOW                              │
└─────────────────────────────────────────────────────────────────────┘

INPUT: Farmer Application
├── Farmer ID
├── GPS Coordinates (lat, lon)
├── Claimed Area (hectares)
├── Claimed Crop Type
├── Planting Date
└── Disaster Claim (optional)
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    STEP 1: DATA RETRIEVAL                            │
│  Fetch from GEE:                                                     │
│  ├── Sentinel-2 imagery (current + historical)                      │
│  ├── Dynamic World land cover                                        │
│  ├── CHIRPS rainfall data                                            │
│  ├── WorldPop population density                                     │
│  └── Sentinel-1 SAR (if flood claim)                                │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    STEP 2: BOUNDARY DETECTION                        │
│  ├── Apply SNIC segmentation to Sentinel-2                          │
│  ├── Extract farm boundary polygon                                   │
│  └── Calculate detected area                                         │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    STEP 3: INDICATOR CALCULATION                     │
│  Calculate all 7 indicators in parallel:                            │
│  ├── [1] Size Discrepancy Score                                     │
│  ├── [2] Crop Mismatch Score                                        │
│  ├── [3] Weather Validation Score                                   │
│  ├── [4] Ghost Farmer Score                                         │
│  ├── [5] Historical Consistency Score                               │
│  ├── [6] Disaster Validation Score                                  │
│  └── [7] Cropland Signal Score                                      │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    STEP 4: TOTAL SCORE CALCULATION                   │
│  ├── Sum all indicator scores                                       │
│  ├── Scale to 0-100                                                 │
│  └── Classify risk level                                            │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    STEP 5: EVIDENCE GENERATION                       │
│  ├── Generate satellite image thumbnails                            │
│  ├── Create boundary overlay maps                                   │
│  ├── Generate NDVI/rainfall charts                                  │
│  └── Compile human-readable report                                  │
└─────────────────────────────────────────────────────────────────────┘
         │
         ▼
OUTPUT: Fraud Assessment
├── Fraud Score (0-100)
├── Risk Level (LOW/MEDIUM/HIGH)
├── Recommendation (APPROVE/REVIEW/REJECT)
├── Individual Indicator Breakdown
└── Evidence Package
```

---

## 3. INDICATOR 1: SIZE DISCREPANCY

### 3.1 What It Detects

**Purpose:** Detect farmers who claim larger farm areas than they actually cultivate.

**Fraud Pattern:** A farmer claims 5 hectares but satellite imagery shows only 2 hectares of cultivated land.

**Why It Matters:** Area inflation is the most common form of agricultural fraud, directly affecting subsidy amounts and insurance payouts.

### 3.2 How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SIZE DISCREPANCY DETECTION                        │
└─────────────────────────────────────────────────────────────────────┘

STEP 1: Get claimed area from farmer application
        Claimed Area = 5.0 hectares

STEP 2: Detect actual farm boundary using SNIC segmentation
        ├── Apply SNIC to Sentinel-2 imagery
        ├── Extract cluster containing farm point
        └── Convert to polygon

STEP 3: Calculate detected area
        Detected Area = 2.1 hectares

STEP 4: Calculate discrepancy percentage
        Discrepancy = |Claimed - Detected| / Claimed × 100
        Discrepancy = |5.0 - 2.1| / 5.0 × 100 = 58%

STEP 5: Apply scoring rules
        ├── 0-15%:   0 points (acceptable tolerance)
        ├── 15-30%:  10 points (minor discrepancy)
        ├── 30-50%:  20 points (significant discrepancy)
        └── >50%:    30 points (major discrepancy - likely fraud)

RESULT: 58% discrepancy → 30 points (HIGH RISK)
```

### 3.3 Scoring Thresholds

| Discrepancy Range | Points | Interpretation                                |
| ----------------- | ------ | --------------------------------------------- |
| 0% - 15%          | 0      | Acceptable (GPS error, boundary uncertainty)  |
| 15% - 30%         | 10     | Minor discrepancy (needs attention)           |
| 30% - 50%         | 20     | Significant discrepancy (likely exaggeration) |
| > 50%             | 30     | Major discrepancy (probable fraud)            |

### 3.4 GEE Implementation

```javascript
/**
 * INDICATOR 1: SIZE DISCREPANCY
 * Compares claimed area with satellite-detected area
 *
 * @param {ee.Geometry.Point} farmPoint - Farm center coordinates
 * @param {number} claimedArea - Farmer's claimed area in hectares
 * @returns {Object} Size discrepancy assessment
 */
function calculateSizeDiscrepancy(farmPoint, claimedArea) {
  // Step 1: Define search region (300m buffer around point)
  var searchRadius = 300;
  var region = farmPoint.buffer(searchRadius);

  // Step 2: Get recent Sentinel-2 imagery
  var s2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(region)
    .filterDate(ee.Date(Date.now()).advance(-30, "day"), ee.Date(Date.now()))
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median()
    .clip(region);

  // Step 3: Apply SNIC segmentation
  var seeds = ee.Algorithms.Image.Segmentation.seedGrid(10);
  var snic = ee.Algorithms.Image.Segmentation.SNIC({
    image: s2.select(["B4", "B3", "B2", "B8"]),
    size: 10,
    compactness: 0,
    connectivity: 8,
    neighborhoodSize: 50,
    seeds: seeds,
  });

  // Step 4: Get cluster at farm point
  var clusterAtPoint = snic.select("clusters").reduceRegion({
    reducer: ee.Reducer.first(),
    geometry: farmPoint,
    scale: 10,
  });
  var clusterId = ee.Number(clusterAtPoint.get("clusters"));

  // Step 5: Create farm mask and convert to polygon
  var farmMask = snic.select("clusters").eq(clusterId);
  var farmVector = farmMask.selfMask().reduceToVectors({
    geometry: region,
    scale: 10,
    geometryType: "polygon",
    maxPixels: 1e8,
  });

  // Step 6: Calculate detected area in hectares
  var detectedBoundary = farmVector.first().geometry();
  var detectedAreaM2 = detectedBoundary.area();
  var detectedAreaHa = detectedAreaM2.divide(10000);

  // Step 7: Calculate discrepancy percentage
  var discrepancy = ee
    .Number(claimedArea)
    .subtract(detectedAreaHa)
    .abs()
    .divide(ee.Number(claimedArea))
    .multiply(100);

  // Step 8: Calculate score based on thresholds
  var score = ee.Algorithms.If(
    discrepancy.lte(15),
    0,
    ee.Algorithms.If(
      discrepancy.lte(30),
      10,
      ee.Algorithms.If(discrepancy.lte(50), 20, 30)
    )
  );

  return {
    indicator: "SIZE_DISCREPANCY",
    claimedArea: claimedArea,
    detectedArea: detectedAreaHa,
    discrepancyPercent: discrepancy,
    score: score,
    maxScore: 30,
    boundary: detectedBoundary,
    evidence: {
      description: ee
        .String("Claimed: ")
        .cat(ee.String(claimedArea))
        .cat(" ha, Detected: ")
        .cat(detectedAreaHa.format("%.2f"))
        .cat(" ha, Discrepancy: ")
        .cat(discrepancy.format("%.1f"))
        .cat("%"),
    },
  };
}
```

### 3.5 GEE Feasibility

| Component          | Available in GEE | Notes                                   |
| ------------------ | ---------------- | --------------------------------------- |
| Sentinel-2 imagery | ✅ Yes           | `COPERNICUS/S2_SR_HARMONIZED`           |
| SNIC segmentation  | ✅ Yes           | `ee.Algorithms.Image.Segmentation.SNIC` |
| Area calculation   | ✅ Yes           | `geometry.area()`                       |
| Vector conversion  | ✅ Yes           | `reduceToVectors()`                     |

**VERDICT: ✅ FULLY IMPLEMENTABLE IN GEE**

---

## 4. INDICATOR 2: CROP TYPE MISMATCH

### 4.1 What It Detects

**Purpose:** Detect farmers who claim to grow high-value crops but actually grow low-value crops (or nothing).

**Fraud Pattern:** A farmer claims to grow maize (higher subsidy) but satellite imagery shows cassava or bare soil.

**Why It Matters:** Different crops have different subsidy rates and insurance premiums. Misrepresenting crop type can result in higher payouts.

### 4.2 How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CROP TYPE MISMATCH DETECTION                      │
└─────────────────────────────────────────────────────────────────────┘

STEP 1: Get claimed crop from farmer application
        Claimed Crop = "maize"

STEP 2: Calculate spectral indices for detected farm area
        ├── NDVI = (NIR - RED) / (NIR + RED)
        ├── EVI = 2.5 × (NIR - RED) / (NIR + 6×RED - 7.5×BLUE + 1)
        └── Get mean values within farm boundary

STEP 3: Apply crop classification rules
        ├── Maize:   NDVI 0.5-0.8, EVI 0.4-0.7
        ├── Rice:    NDVI 0.3-0.6, EVI 0.2-0.5
        ├── Cassava: NDVI 0.4-0.7, EVI 0.3-0.6
        └── etc.

STEP 4: Compare detected crop with claimed crop
        Detected Crop = "cassava"
        Match = FALSE

STEP 5: Apply scoring rules
        ├── Match:              0 points
        ├── Similar crop:       15 points (e.g., sorghum vs millet)
        └── Different crop:     30 points

RESULT: Maize claimed, Cassava detected → 30 points (HIGH RISK)
```

### 4.3 Crop Classification Rules

| Crop      | NDVI Range | EVI Range | Growing Season |
| --------- | ---------- | --------- | -------------- |
| Maize     | 0.5 - 0.8  | 0.4 - 0.7 | Mar-Aug        |
| Rice      | 0.3 - 0.6  | 0.2 - 0.5 | Jun-Nov        |
| Cassava   | 0.4 - 0.7  | 0.3 - 0.6 | Year-round     |
| Sorghum   | 0.4 - 0.7  | 0.3 - 0.6 | Mar-Sep        |
| Beans     | 0.3 - 0.6  | 0.2 - 0.5 | Mar-Jun        |
| Millet    | 0.3 - 0.5  | 0.2 - 0.4 | Jun-Oct        |
| Bare Soil | < 0.2      | < 0.15    | N/A            |

### 4.4 Scoring Thresholds

| Scenario            | Points | Interpretation                        |
| ------------------- | ------ | ------------------------------------- |
| Exact match         | 0      | Claimed crop matches detected crop    |
| Similar crop family | 15     | Minor mismatch (e.g., sorghum/millet) |
| Different crop      | 30     | Major mismatch (likely fraud)         |
| Bare soil/no crop   | 30     | No cultivation detected               |

### 4.5 GEE Implementation

```javascript
/**
 * INDICATOR 2: CROP TYPE MISMATCH
 * Compares claimed crop with satellite-detected crop type
 *
 * @param {ee.Geometry} farmBoundary - Detected farm boundary
 * @param {string} claimedCrop - Farmer's claimed crop type
 * @param {string} plantingDate - Planting date for seasonal adjustment
 * @returns {Object} Crop mismatch assessment
 */
function calculateCropMismatch(farmBoundary, claimedCrop, plantingDate) {
  // Step 1: Get growing season imagery (2-4 months after planting)
  var startDate = ee.Date(plantingDate).advance(2, "month");
  var endDate = ee.Date(plantingDate).advance(4, "month");

  var s2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(farmBoundary)
    .filterDate(startDate, endDate)
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median();

  // Step 2: Calculate NDVI
  var ndvi = s2.normalizedDifference(["B8", "B4"]).rename("NDVI");

  // Step 3: Calculate EVI
  var evi = s2
    .expression("2.5 * ((NIR - RED) / (NIR + 6 * RED - 7.5 * BLUE + 1))", {
      NIR: s2.select("B8"),
      RED: s2.select("B4"),
      BLUE: s2.select("B2"),
    })
    .rename("EVI");

  // Step 4: Get mean values within farm boundary
  var stats = ndvi.addBands(evi).reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 10,
  });

  var ndviMean = ee.Number(stats.get("NDVI"));
  var eviMean = ee.Number(stats.get("EVI"));

  // Step 5: Classify crop based on spectral rules
  var detectedCrop = classifyCropByRules(ndviMean, eviMean);

  // Step 6: Compare with claimed crop
  var cropMatch = ee.String(detectedCrop).equals(ee.String(claimedCrop));
  var similarCrop = checkSimilarCrop(claimedCrop, detectedCrop);

  // Step 7: Calculate score
  var score = ee.Algorithms.If(
    cropMatch,
    0,
    ee.Algorithms.If(similarCrop, 15, 30)
  );

  return {
    indicator: "CROP_MISMATCH",
    claimedCrop: claimedCrop,
    detectedCrop: detectedCrop,
    cropMatch: cropMatch,
    ndvi: ndviMean,
    evi: eviMean,
    score: score,
    maxScore: 30,
    evidence: {
      description: ee
        .String("Claimed: ")
        .cat(claimedCrop)
        .cat(", Detected: ")
        .cat(detectedCrop)
        .cat(", NDVI: ")
        .cat(ndviMean.format("%.2f")),
    },
  };
}

/**
 * Classify crop based on NDVI/EVI rules
 */
function classifyCropByRules(ndvi, evi) {
  return ee.Algorithms.If(
    ndvi.lt(0.2),
    "bare_soil",
    ee.Algorithms.If(
      ndvi.gte(0.5).and(ndvi.lte(0.8)).and(evi.gte(0.4)),
      "maize",
      ee.Algorithms.If(
        ndvi.gte(0.3).and(ndvi.lte(0.6)).and(evi.lt(0.4)),
        "rice",
        ee.Algorithms.If(ndvi.gte(0.4).and(ndvi.lte(0.7)), "cassava", "unknown")
      )
    )
  );
}

/**
 * Check if two crops are in the same family
 */
function checkSimilarCrop(crop1, crop2) {
  var cerealFamily = ["maize", "sorghum", "millet", "rice"];
  var legumesFamily = ["beans", "groundnuts", "cowpeas"];

  // Check if both crops are in the same family
  var crop1InCereals = cerealFamily.indexOf(crop1) >= 0;
  var crop2InCereals = cerealFamily.indexOf(crop2) >= 0;
  var crop1InLegumes = legumesFamily.indexOf(crop1) >= 0;
  var crop2InLegumes = legumesFamily.indexOf(crop2) >= 0;

  return (
    (crop1InCereals && crop2InCereals) || (crop1InLegumes && crop2InLegumes)
  );
}
```

### 4.6 GEE Feasibility

| Component          | Available in GEE | Notes                         |
| ------------------ | ---------------- | ----------------------------- |
| Sentinel-2 imagery | ✅ Yes           | `COPERNICUS/S2_SR_HARMONIZED` |
| NDVI calculation   | ✅ Yes           | `normalizedDifference()`      |
| EVI calculation    | ✅ Yes           | `expression()`                |
| Zonal statistics   | ✅ Yes           | `reduceRegion()`              |

**VERDICT: ✅ FULLY IMPLEMENTABLE IN GEE**

---

## 5. INDICATOR 3: WEATHER VALIDATION

### 5.1 What It Detects

**Purpose:** Validate that weather conditions were suitable for the claimed crop during the growing season.

**Fraud Pattern:** A farmer claims successful maize harvest, but rainfall data shows severe drought during the growing season (making successful harvest impossible).

**Why It Matters:** Validates the plausibility of crop claims and helps detect false harvest claims.

### 5.2 How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WEATHER VALIDATION                                │
└─────────────────────────────────────────────────────────────────────┘

STEP 1: Get crop water requirements
        Maize requires: 450-600mm during growing season

STEP 2: Get actual rainfall from CHIRPS
        ├── Filter to growing season dates
        ├── Sum total precipitation
        └── Actual rainfall = 180mm

STEP 3: Compare with requirements
        Required: 450mm minimum
        Actual: 180mm
        Deficit: 60% below requirement

STEP 4: Apply scoring rules
        ├── Adequate rainfall:     0 points
        ├── Mild deficit (10-30%): 10 points
        └── Severe deficit (>30%): 20 points

RESULT: 60% deficit → 20 points (SUSPICIOUS)
```

### 5.3 Crop Water Requirements

| Crop    | Min Rainfall (mm) | Optimal Rainfall (mm) | Growing Period |
| ------- | ----------------- | --------------------- | -------------- |
| Maize   | 450               | 600-800               | 90-120 days    |
| Rice    | 1000              | 1200-1500             | 120-150 days   |
| Cassava | 500               | 1000-1500             | 270-365 days   |
| Sorghum | 300               | 450-650               | 90-120 days    |
| Beans   | 300               | 400-500               | 60-90 days     |
| Millet  | 250               | 350-500               | 60-90 days     |

### 5.4 Scoring Thresholds

| Rainfall Status                | Points | Interpretation              |
| ------------------------------ | ------ | --------------------------- |
| Adequate (≥90% of requirement) | 0      | Normal conditions           |
| Mild deficit (70-90%)          | 10     | Possible stress             |
| Severe deficit (<70%)          | 20     | Unlikely successful harvest |

### 5.5 GEE Implementation

```javascript
/**
 * INDICATOR 3: WEATHER VALIDATION
 * Validates rainfall adequacy for claimed crop
 *
 * @param {ee.Geometry} farmBoundary - Farm boundary
 * @param {string} claimedCrop - Farmer's claimed crop
 * @param {string} plantingDate - Planting date
 * @returns {Object} Weather validation assessment
 */
function calculateWeatherValidation(farmBoundary, claimedCrop, plantingDate) {
  // Step 1: Define growing season (crop-specific duration)
  var cropDurations = {
    maize: 120,
    rice: 150,
    cassava: 300,
    sorghum: 110,
    beans: 80,
    millet: 80,
  };
  var duration = cropDurations[claimedCrop] || 120;

  var startDate = ee.Date(plantingDate);
  var endDate = startDate.advance(duration, "day");

  // Step 2: Get CHIRPS rainfall data
  var chirps = ee
    .ImageCollection("UCSB-CHG/CHIRPS/DAILY")
    .filterBounds(farmBoundary)
    .filterDate(startDate, endDate);

  // Step 3: Calculate total rainfall
  var totalRainfall = chirps.select("precipitation").sum();
  var rainfallStats = totalRainfall.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 5000,
  });
  var actualRainfall = ee.Number(rainfallStats.get("precipitation"));

  // Step 4: Get crop water requirement
  var cropRequirements = {
    maize: 450,
    rice: 1000,
    cassava: 500,
    sorghum: 300,
    beans: 300,
    millet: 250,
  };
  var requiredRainfall = ee.Number(cropRequirements[claimedCrop] || 400);

  // Step 5: Calculate deficit percentage
  var rainfallRatio = actualRainfall.divide(requiredRainfall);

  // Step 6: Calculate score
  var score = ee.Algorithms.If(
    rainfallRatio.gte(0.9),
    0,
    ee.Algorithms.If(rainfallRatio.gte(0.7), 10, 20)
  );

  return {
    indicator: "WEATHER_VALIDATION",
    actualRainfall: actualRainfall,
    requiredRainfall: requiredRainfall,
    rainfallRatio: rainfallRatio,
    score: score,
    maxScore: 20,
    evidence: {
      description: ee
        .String("Required: ")
        .cat(requiredRainfall.format("%.0f"))
        .cat("mm, Actual: ")
        .cat(actualRainfall.format("%.0f"))
        .cat("mm (")
        .cat(rainfallRatio.multiply(100).format("%.0f"))
        .cat("%)"),
    },
  };
}
```

### 5.6 GEE Feasibility

| Component          | Available in GEE | Notes                   |
| ------------------ | ---------------- | ----------------------- |
| CHIRPS rainfall    | ✅ Yes           | `UCSB-CHG/CHIRPS/DAILY` |
| Temporal filtering | ✅ Yes           | `filterDate()`          |
| Sum calculation    | ✅ Yes           | `sum()`                 |
| Zonal statistics   | ✅ Yes           | `reduceRegion()`        |

**VERDICT: ✅ FULLY IMPLEMENTABLE IN GEE**

---

## 6. INDICATOR 4: GHOST FARMER DETECTION

### 6.1 What It Detects

**Purpose:** Detect "ghost farmers" - fake farmer registrations in uninhabited or inaccessible areas.

**Fraud Pattern:** A fraudster registers multiple fake farms in remote, uninhabited areas to collect subsidies.

**Why It Matters:** Ghost farmer fraud is a significant problem in subsidy programs, with fake registrations in areas where no one actually lives or farms.

### 6.2 How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GHOST FARMER DETECTION                            │
└─────────────────────────────────────────────────────────────────────┘

STEP 1: Get farm location
        Coordinates: [-1.2921, 36.8219]

STEP 2: Get population density from WorldPop
        ├── Query WorldPop dataset
        └── Get population per km² at location

STEP 3: Evaluate population density
        Population density = 0.5 people/km²
        Threshold = 5 people/km²

STEP 4: Apply scoring rules
        ├── Populated (>10/km²):     0 points
        ├── Sparse (5-10/km²):       10 points
        └── Uninhabited (<5/km²):    20 points

RESULT: 0.5 people/km² → 20 points (GHOST FARMER RISK)
```

### 6.3 Scoring Thresholds

| Population Density | Points | Interpretation                         |
| ------------------ | ------ | -------------------------------------- |
| > 10 people/km²    | 0      | Normal populated area                  |
| 5-10 people/km²    | 10     | Sparse population (verify)             |
| < 5 people/km²     | 20     | Likely uninhabited (ghost farmer risk) |

### 6.4 GEE Implementation

```javascript
/**
 * INDICATOR 4: GHOST FARMER DETECTION
 * Checks if farm is in a populated area
 *
 * @param {ee.Geometry.Point} farmPoint - Farm center coordinates
 * @returns {Object} Ghost farmer assessment
 */
function calculateGhostFarmer(farmPoint) {
  // Step 1: Get WorldPop population density
  var worldpop = ee
    .ImageCollection("WorldPop/GP/100m/pop")
    .filterBounds(farmPoint)
    .sort("system:time_start", false)
    .first();

  // Step 2: Get population at farm location (5km buffer for context)
  var region = farmPoint.buffer(5000);
  var popStats = worldpop.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: region,
    scale: 100,
  });

  // WorldPop gives people per 100m pixel, convert to per km²
  var popDensity = ee.Number(popStats.get("population")).multiply(100);

  // Step 3: Calculate score based on population density
  var score = ee.Algorithms.If(
    popDensity.gt(10),
    0,
    ee.Algorithms.If(popDensity.gte(5), 10, 20)
  );

  return {
    indicator: "GHOST_FARMER",
    populationDensity: popDensity,
    score: score,
    maxScore: 20,
    evidence: {
      description: ee
        .String("Population density: ")
        .cat(popDensity.format("%.1f"))
        .cat(" people/km²"),
    },
  };
}
```

### 6.5 GEE Feasibility

| Component       | Available in GEE | Notes                  |
| --------------- | ---------------- | ---------------------- |
| WorldPop data   | ✅ Yes           | `WorldPop/GP/100m/pop` |
| Point query     | ✅ Yes           | `reduceRegion()`       |
| Buffer analysis | ✅ Yes           | `buffer()`             |

**VERDICT: ✅ FULLY IMPLEMENTABLE IN GEE**

---

## 7. INDICATOR 5: HISTORICAL CONSISTENCY

### 7.1 What It Detects

**Purpose:** Detect sudden, unexplained changes in land use that may indicate fraud.

**Fraud Pattern:** A location that was forest or urban area 5 years ago is now claimed as established farmland.

**Why It Matters:** Legitimate farms typically show consistent agricultural activity over time. Sudden changes may indicate false claims.

### 7.2 How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HISTORICAL CONSISTENCY CHECK                      │
└─────────────────────────────────────────────────────────────────────┘

STEP 1: Get current NDVI
        Current NDVI = 0.65 (healthy vegetation)

STEP 2: Get historical NDVI (5 years ago)
        Historical NDVI = 0.15 (bare/urban)

STEP 3: Calculate change
        NDVI Change = |0.65 - 0.15| = 0.50

STEP 4: Apply scoring rules
        ├── Stable (change < 0.15):     0 points
        ├── Minor change (0.15-0.30):   8 points
        └── Major change (> 0.30):      15 points

RESULT: 0.50 change → 15 points (SUSPICIOUS)
```

### 7.3 Scoring Thresholds

| NDVI Change | Points | Interpretation                     |
| ----------- | ------ | ---------------------------------- |
| < 0.15      | 0      | Stable land use                    |
| 0.15 - 0.30 | 8      | Minor change (seasonal variation)  |
| > 0.30      | 15     | Major change (land use conversion) |

### 7.4 GEE Implementation

```javascript
/**
 * INDICATOR 5: HISTORICAL CONSISTENCY
 * Compares current vs historical vegetation patterns
 *
 * @param {ee.Geometry} farmBoundary - Farm boundary
 * @returns {Object} Historical consistency assessment
 */
function calculateHistoricalConsistency(farmBoundary) {
  // Step 1: Get current year imagery
  var currentYear = ee.Date(Date.now()).get("year");
  var currentS2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(farmBoundary)
    .filterDate(
      ee.Date.fromYMD(currentYear, 6, 1),
      ee.Date.fromYMD(currentYear, 9, 30)
    )
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median();

  var currentNDVI = currentS2.normalizedDifference(["B8", "B4"]);

  // Step 2: Get historical imagery (5 years ago)
  var historicalYear = currentYear.subtract(5);
  var historicalS2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(farmBoundary)
    .filterDate(
      ee.Date.fromYMD(historicalYear, 6, 1),
      ee.Date.fromYMD(historicalYear, 9, 30)
    )
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median();

  var historicalNDVI = historicalS2.normalizedDifference(["B8", "B4"]);

  // Step 3: Calculate mean NDVI values
  var currentStats = currentNDVI.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 10,
  });
  var historicalStats = historicalNDVI.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 10,
  });

  var currentMean = ee.Number(currentStats.get("nd"));
  var historicalMean = ee.Number(historicalStats.get("nd"));

  // Step 4: Calculate absolute change
  var ndviChange = currentMean.subtract(historicalMean).abs();

  // Step 5: Calculate score
  var score = ee.Algorithms.If(
    ndviChange.lt(0.15),
    0,
    ee.Algorithms.If(ndviChange.lt(0.3), 8, 15)
  );

  return {
    indicator: "HISTORICAL_CONSISTENCY",
    currentNDVI: currentMean,
    historicalNDVI: historicalMean,
    ndviChange: ndviChange,
    score: score,
    maxScore: 15,
    evidence: {
      description: ee
        .String("Current NDVI: ")
        .cat(currentMean.format("%.2f"))
        .cat(", Historical NDVI: ")
        .cat(historicalMean.format("%.2f"))
        .cat(", Change: ")
        .cat(ndviChange.format("%.2f")),
    },
  };
}
```

### 7.5 GEE Feasibility

| Component             | Available in GEE | Notes                    |
| --------------------- | ---------------- | ------------------------ |
| Sentinel-2 archive    | ✅ Yes           | Data from 2015+          |
| Historical comparison | ✅ Yes           | `filterDate()`           |
| NDVI calculation      | ✅ Yes           | `normalizedDifference()` |

**VERDICT: ✅ FULLY IMPLEMENTABLE IN GEE**

---

## 8. INDICATOR 6: DISASTER CLAIM VALIDATION

### 8.1 What It Detects

**Purpose:** Validate disaster claims (flood, drought) against actual satellite-observed conditions.

**Fraud Pattern:** A farmer claims flood damage, but satellite imagery shows no flooding occurred.

**Why It Matters:** Disaster claims trigger higher payouts. False disaster claims are a significant fraud vector.

### 8.2 How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DISASTER CLAIM VALIDATION                         │
└─────────────────────────────────────────────────────────────────────┘

FLOOD CLAIM VALIDATION:
STEP 1: Get Sentinel-1 SAR data for claimed flood period
STEP 2: Calculate water index (VV/VH ratio)
STEP 3: Detect flood extent
STEP 4: Check if farm was actually flooded

DROUGHT CLAIM VALIDATION:
STEP 1: Get CHIRPS rainfall for claimed drought period
STEP 2: Calculate rainfall deficit
STEP 3: Check if drought actually occurred

SCORING:
├── Disaster confirmed:     0 points
├── Partial evidence:       5 points
└── No evidence of disaster: 10 points
```

### 8.3 GEE Implementation

```javascript
/**
 * INDICATOR 6: DISASTER CLAIM VALIDATION
 * Validates flood/drought claims against satellite data
 *
 * @param {ee.Geometry} farmBoundary - Farm boundary
 * @param {string} disasterType - 'flood' or 'drought'
 * @param {string} disasterDate - Claimed disaster date
 * @returns {Object} Disaster validation assessment
 */
function calculateDisasterValidation(farmBoundary, disasterType, disasterDate) {
  if (!disasterType) {
    return {
      indicator: "DISASTER_VALIDATION",
      score: 0,
      maxScore: 10,
      evidence: { description: "No disaster claim" },
    };
  }

  if (disasterType === "flood") {
    return validateFloodClaim(farmBoundary, disasterDate);
  } else if (disasterType === "drought") {
    return validateDroughtClaim(farmBoundary, disasterDate);
  }
}

/**
 * Validate flood claim using Sentinel-1 SAR
 */
function validateFloodClaim(farmBoundary, disasterDate) {
  var eventDate = ee.Date(disasterDate);

  // Get SAR imagery during claimed flood
  var duringFlood = ee
    .ImageCollection("COPERNICUS/S1_GRD")
    .filterBounds(farmBoundary)
    .filterDate(eventDate.advance(-7, "day"), eventDate.advance(7, "day"))
    .filter(ee.Filter.eq("instrumentMode", "IW"))
    .select("VV")
    .mean();

  // Get SAR imagery before flood (baseline)
  var beforeFlood = ee
    .ImageCollection("COPERNICUS/S1_GRD")
    .filterBounds(farmBoundary)
    .filterDate(eventDate.advance(-60, "day"), eventDate.advance(-30, "day"))
    .filter(ee.Filter.eq("instrumentMode", "IW"))
    .select("VV")
    .mean();

  // Calculate change (flooding causes VV to decrease)
  var vvChange = duringFlood.subtract(beforeFlood);

  // Get mean change within farm
  var changeStats = vvChange.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 10,
  });
  var meanChange = ee.Number(changeStats.get("VV"));

  // Flooding typically shows VV decrease of > 3 dB
  var floodDetected = meanChange.lt(-3);

  var score = ee.Algorithms.If(floodDetected, 0, 10);

  return {
    indicator: "DISASTER_VALIDATION",
    disasterType: "flood",
    vvChange: meanChange,
    disasterConfirmed: floodDetected,
    score: score,
    maxScore: 10,
    evidence: {
      description: ee
        .String("VV change: ")
        .cat(meanChange.format("%.1f"))
        .cat(" dB. Flood ")
        .cat(ee.Algorithms.If(floodDetected, "CONFIRMED", "NOT DETECTED")),
    },
  };
}

/**
 * Validate drought claim using CHIRPS rainfall
 */
function validateDroughtClaim(farmBoundary, disasterDate) {
  var eventDate = ee.Date(disasterDate);

  // Get rainfall for claimed drought period (3 months)
  var droughtPeriod = ee
    .ImageCollection("UCSB-CHG/CHIRPS/DAILY")
    .filterBounds(farmBoundary)
    .filterDate(eventDate.advance(-90, "day"), eventDate)
    .select("precipitation")
    .sum();

  // Get historical average for same period
  var historicalYears = ee.List.sequence(2015, 2023);
  var historicalAvg = historicalYears.map(function (year) {
    var startDate = ee.Date.fromYMD(year, eventDate.get("month"), 1).advance(
      -90,
      "day"
    );
    var endDate = ee.Date.fromYMD(
      year,
      eventDate.get("month"),
      eventDate.get("day")
    );
    return ee
      .ImageCollection("UCSB-CHG/CHIRPS/DAILY")
      .filterBounds(farmBoundary)
      .filterDate(startDate, endDate)
      .select("precipitation")
      .sum();
  });

  var avgRainfall = ee.ImageCollection(historicalAvg).mean();

  // Calculate deficit
  var currentStats = droughtPeriod.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 5000,
  });
  var avgStats = avgRainfall.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 5000,
  });

  var currentRainfall = ee.Number(currentStats.get("precipitation"));
  var avgRainfallVal = ee.Number(avgStats.get("precipitation"));
  var deficit = ee.Number(1).subtract(currentRainfall.divide(avgRainfallVal));

  // Drought = >40% below average
  var droughtDetected = deficit.gt(0.4);

  var score = ee.Algorithms.If(droughtDetected, 0, 10);

  return {
    indicator: "DISASTER_VALIDATION",
    disasterType: "drought",
    currentRainfall: currentRainfall,
    averageRainfall: avgRainfallVal,
    deficit: deficit,
    disasterConfirmed: droughtDetected,
    score: score,
    maxScore: 10,
  };
}
```

### 8.4 GEE Feasibility

| Component         | Available in GEE | Notes                        |
| ----------------- | ---------------- | ---------------------------- |
| Sentinel-1 SAR    | ✅ Yes           | `COPERNICUS/S1_GRD`          |
| CHIRPS rainfall   | ✅ Yes           | `UCSB-CHG/CHIRPS/DAILY`      |
| Flood detection   | ✅ Yes           | VV backscatter analysis      |
| Drought detection | ✅ Yes           | Rainfall deficit calculation |

**VERDICT: ✅ FULLY IMPLEMENTABLE IN GEE**

---

## 9. INDICATOR 7: CROPLAND SIGNAL

### 9.1 What It Detects

**Purpose:** Verify that the claimed location actually shows cropland characteristics.

**Fraud Pattern:** A farmer claims a location that is actually forest, water, or urban area.

**Why It Matters:** Basic validation that the location is actually agricultural land.

### 9.2 How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CROPLAND SIGNAL DETECTION                         │
└─────────────────────────────────────────────────────────────────────┘

STEP 1: Get Dynamic World land cover classification
        ├── Query Dynamic World at farm location
        └── Get cropland probability

STEP 2: Get vegetation signal
        ├── Calculate NDVI
        └── Check for agricultural signature

STEP 3: Evaluate cropland probability
        Cropland probability = 15%
        NDVI = 0.12

STEP 4: Apply scoring rules
        ├── High cropland signal (>60% + NDVI>0.3):  0 points
        ├── Medium signal (30-60%):                   5 points
        └── Low signal (<30%):                        10 points

RESULT: 15% cropland, 0.12 NDVI → 10 points (NOT CROPLAND)
```

### 9.3 Scoring Thresholds

| Cropland Signal                    | Points | Interpretation     |
| ---------------------------------- | ------ | ------------------ |
| High (>60% probability + NDVI>0.3) | 0      | Confirmed cropland |
| Medium (30-60% probability)        | 5      | Possible cropland  |
| Low (<30% probability)             | 10     | Unlikely cropland  |

### 9.4 GEE Implementation

```javascript
/**
 * INDICATOR 7: CROPLAND SIGNAL
 * Validates that location shows cropland characteristics
 *
 * @param {ee.Geometry} farmBoundary - Farm boundary
 * @returns {Object} Cropland signal assessment
 */
function calculateCroplandSignal(farmBoundary) {
  // Step 1: Get Dynamic World land cover
  var dynamicWorld = ee
    .ImageCollection("GOOGLE/DYNAMICWORLD/V1")
    .filterBounds(farmBoundary)
    .filterDate(ee.Date(Date.now()).advance(-30, "day"), ee.Date(Date.now()))
    .select("crops")
    .mean();

  // Step 2: Get cropland probability
  var croplandStats = dynamicWorld.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 10,
  });
  var croplandProb = ee.Number(croplandStats.get("crops"));

  // Step 3: Get NDVI as secondary check
  var s2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(farmBoundary)
    .filterDate(ee.Date(Date.now()).advance(-30, "day"), ee.Date(Date.now()))
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median();

  var ndvi = s2.normalizedDifference(["B8", "B4"]);
  var ndviStats = ndvi.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 10,
  });
  var ndviMean = ee.Number(ndviStats.get("nd"));

  // Step 4: Calculate score
  var highSignal = croplandProb.gt(0.6).and(ndviMean.gt(0.3));
  var mediumSignal = croplandProb.gt(0.3);

  var score = ee.Algorithms.If(
    highSignal,
    0,
    ee.Algorithms.If(mediumSignal, 5, 10)
  );

  return {
    indicator: "CROPLAND_SIGNAL",
    croplandProbability: croplandProb,
    ndvi: ndviMean,
    score: score,
    maxScore: 10,
    evidence: {
      description: ee
        .String("Cropland probability: ")
        .cat(croplandProb.multiply(100).format("%.0f"))
        .cat("%, NDVI: ")
        .cat(ndviMean.format("%.2f")),
    },
  };
}
```

### 9.5 GEE Feasibility

| Component            | Available in GEE | Notes                    |
| -------------------- | ---------------- | ------------------------ |
| Dynamic World        | ✅ Yes           | `GOOGLE/DYNAMICWORLD/V1` |
| Cropland probability | ✅ Yes           | `crops` band             |
| NDVI calculation     | ✅ Yes           | `normalizedDifference()` |

**VERDICT: ✅ FULLY IMPLEMENTABLE IN GEE**

---

## 10. SCORING ALGORITHM

### 10.1 Total Score Calculation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SCORING ALGORITHM                                 │
└─────────────────────────────────────────────────────────────────────┘

STEP 1: Collect all indicator scores
┌─────────────────────────────────────────────────────────────────────┐
│  Indicator                    │ Score │ Max   │ Weight             │
├───────────────────────────────┼───────┼───────┼────────────────────┤
│  1. Size Discrepancy          │  20   │  30   │ 22.2%              │
│  2. Crop Type Mismatch        │   0   │  30   │ 22.2%              │
│  3. Weather Validation        │  10   │  20   │ 14.8%              │
│  4. Ghost Farmer Detection    │   0   │  20   │ 14.8%              │
│  5. Historical Consistency    │   8   │  15   │ 11.1%              │
│  6. Disaster Claim Validation │   0   │  10   │  7.4%              │
│  7. Cropland Signal           │   0   │  10   │  7.4%              │
├───────────────────────────────┼───────┼───────┼────────────────────┤
│  TOTAL                        │  38   │ 135   │ 100%               │
└─────────────────────────────────────────────────────────────────────┘

STEP 2: Calculate scaled score
        Scaled Score = (Raw Score / Max Score) × 100
        Scaled Score = (38 / 135) × 100 = 28.1

STEP 3: Classify risk level
        ├── 0-39:   LOW RISK    → APPROVE
        ├── 40-69:  MEDIUM RISK → MANUAL REVIEW
        └── 70-100: HIGH RISK   → REJECT

RESULT: Score 28.1 → LOW RISK → APPROVE
```

### 10.2 GEE Implementation

```javascript
/**
 * MASTER FRAUD SCORING FUNCTION
 * Calculates total fraud score from all indicators
 *
 * @param {Object} indicators - All calculated indicator results
 * @returns {Object} Final fraud assessment
 */
function calculateTotalFraudScore(indicators) {
  // Step 1: Sum all indicator scores
  var totalRawScore = ee
    .Number(0)
    .add(indicators.sizeDiscrepancy.score)
    .add(indicators.cropMismatch.score)
    .add(indicators.weatherValidation.score)
    .add(indicators.ghostFarmer.score)
    .add(indicators.historicalConsistency.score)
    .add(indicators.disasterValidation.score)
    .add(indicators.croplandSignal.score);

  // Step 2: Maximum possible score
  var maxScore = 135;

  // Step 3: Scale to 0-100
  var scaledScore = totalRawScore.divide(maxScore).multiply(100);

  // Step 4: Determine risk level and recommendation
  var riskLevel = ee.Algorithms.If(
    scaledScore.gte(70),
    "HIGH",
    ee.Algorithms.If(scaledScore.gte(40), "MEDIUM", "LOW")
  );

  var recommendation = ee.Algorithms.If(
    scaledScore.gte(70),
    "REJECT",
    ee.Algorithms.If(scaledScore.gte(40), "MANUAL_REVIEW", "APPROVE")
  );

  return {
    rawScore: totalRawScore,
    maxScore: maxScore,
    scaledScore: scaledScore,
    riskLevel: riskLevel,
    recommendation: recommendation,
    indicators: {
      sizeDiscrepancy: indicators.sizeDiscrepancy,
      cropMismatch: indicators.cropMismatch,
      weatherValidation: indicators.weatherValidation,
      ghostFarmer: indicators.ghostFarmer,
      historicalConsistency: indicators.historicalConsistency,
      disasterValidation: indicators.disasterValidation,
      croplandSignal: indicators.croplandSignal,
    },
  };
}
```

---

## 11. RISK CLASSIFICATION

### 11.1 Risk Levels

| Risk Level | Score Range | Recommendation | Action                    |
| ---------- | ----------- | -------------- | ------------------------- |
| **LOW**    | 0 - 39      | APPROVE        | Auto-approve application  |
| **MEDIUM** | 40 - 69     | MANUAL_REVIEW  | Flag for human review     |
| **HIGH**   | 70 - 100    | REJECT         | Auto-reject with evidence |

### 11.2 Decision Matrix

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DECISION MATRIX                                   │
└─────────────────────────────────────────────────────────────────────┘

LOW RISK (0-39):
├── Typical profile: Minor size discrepancy only
├── Action: Auto-approve
├── Processing: Immediate
└── Human review: Not required

MEDIUM RISK (40-69):
├── Typical profile: Multiple minor flags OR one major flag
├── Action: Queue for manual review
├── Processing: Within 24-48 hours
└── Human review: Required before decision

HIGH RISK (70-100):
├── Typical profile: Multiple major flags
├── Action: Auto-reject with detailed evidence
├── Processing: Immediate rejection
└── Human review: Optional appeal process
```

---

## 12. EVIDENCE PACKAGE GENERATION

### 12.1 Evidence Components

Each verification generates an evidence package containing:

| Component             | Description                          | Format        |
| --------------------- | ------------------------------------ | ------------- |
| Satellite Image       | RGB composite of farm area           | PNG/JPEG      |
| Boundary Overlay      | Detected boundary on satellite image | GeoJSON + PNG |
| NDVI Map              | Vegetation health visualization      | PNG           |
| Rainfall Chart        | Precipitation over growing season    | Chart image   |
| Historical Comparison | Side-by-side current vs historical   | PNG           |
| Indicator Summary     | All indicator scores and details     | JSON          |
| Human-Readable Report | Summary for reviewers                | PDF/HTML      |

### 12.2 GEE Implementation

```javascript
/**
 * Generate evidence package for verification
 *
 * @param {ee.Geometry} farmBoundary - Detected farm boundary
 * @param {Object} indicators - All indicator results
 * @returns {Object} Evidence package with URLs
 */
function generateEvidencePackage(farmBoundary, indicators) {
  var region = farmBoundary.bounds().buffer(100);

  // 1. Generate RGB satellite image
  var s2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(region)
    .filterDate(ee.Date(Date.now()).advance(-30, "day"), ee.Date(Date.now()))
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median();

  var rgbVis = { bands: ["B4", "B3", "B2"], min: 0, max: 3000 };
  var satelliteThumb = s2.getThumbURL({
    region: region,
    dimensions: 512,
    format: "png",
    bands: ["B4", "B3", "B2"],
    min: 0,
    max: 3000,
  });

  // 2. Generate NDVI visualization
  var ndvi = s2.normalizedDifference(["B8", "B4"]);
  var ndviThumb = ndvi.getThumbURL({
    region: region,
    dimensions: 512,
    format: "png",
    palette: ["red", "yellow", "green"],
    min: 0,
    max: 1,
  });

  // 3. Generate boundary GeoJSON
  var boundaryGeoJSON = farmBoundary.toGeoJSON();

  return {
    satelliteImageUrl: satelliteThumb,
    ndviMapUrl: ndviThumb,
    boundaryGeoJSON: boundaryGeoJSON,
    indicatorSummary: indicators,
    generatedAt: ee.Date(Date.now()).format("YYYY-MM-dd HH:mm:ss"),
  };
}
```

---

## 13. END-TO-END WORKFLOW EXAMPLE

### 13.1 Complete Verification Example

```javascript
/**
 * COMPLETE FARM VERIFICATION WORKFLOW
 * End-to-end example processing a single farm
 */
function verifyFarm(
  farmerId,
  lat,
  lon,
  claimedArea,
  claimedCrop,
  plantingDate,
  disasterClaim
) {
  // Step 1: Create farm point geometry
  var farmPoint = ee.Geometry.Point([lon, lat]);

  // Step 2: Detect farm boundary
  var boundaryResult = calculateSizeDiscrepancy(farmPoint, claimedArea);
  var farmBoundary = boundaryResult.boundary;

  // Step 3: Calculate all indicators (parallel in GEE)
  var indicators = {
    sizeDiscrepancy: boundaryResult,
    cropMismatch: calculateCropMismatch(
      farmBoundary,
      claimedCrop,
      plantingDate
    ),
    weatherValidation: calculateWeatherValidation(
      farmBoundary,
      claimedCrop,
      plantingDate
    ),
    ghostFarmer: calculateGhostFarmer(farmPoint),
    historicalConsistency: calculateHistoricalConsistency(farmBoundary),
    disasterValidation: calculateDisasterValidation(
      farmBoundary,
      disasterClaim,
      plantingDate
    ),
    croplandSignal: calculateCroplandSignal(farmBoundary),
  };

  // Step 4: Calculate total fraud score
  var fraudAssessment = calculateTotalFraudScore(indicators);

  // Step 5: Generate evidence package
  var evidence = generateEvidencePackage(farmBoundary, indicators);

  // Step 6: Compile final result
  return {
    farmerId: farmerId,
    verificationId: "VER-" + Date.now(),
    timestamp: new Date().toISOString(),
    coordinates: { lat: lat, lon: lon },
    claimedArea: claimedArea,
    detectedArea: boundaryResult.detectedArea,
    claimedCrop: claimedCrop,
    fraudScore: fraudAssessment.scaledScore,
    riskLevel: fraudAssessment.riskLevel,
    recommendation: fraudAssessment.recommendation,
    indicators: fraudAssessment.indicators,
    evidence: evidence,
  };
}

// Example usage:
var result = verifyFarm(
  "FRM-12345", // farmerId
  -1.2921, // latitude
  36.8219, // longitude
  2.5, // claimedArea (hectares)
  "maize", // claimedCrop
  "2024-03-15", // plantingDate
  null // disasterClaim (null = no claim)
);

print("Verification Result:", result);
```

---

## 14. GEE FEASIBILITY SUMMARY

### 14.1 Complete Feature Availability Matrix

| Feature                    | GEE Available | Data Source         | Notes                 |
| -------------------------- | ------------- | ------------------- | --------------------- |
| **Boundary Detection**     | ✅ Yes        | Sentinel-2 + SNIC   | 10m resolution        |
| **Crop Classification**    | ✅ Yes        | Sentinel-2 NDVI/EVI | Rules-based           |
| **Area Calculation**       | ✅ Yes        | `geometry.area()`   | Accurate to 10m       |
| **Weather Validation**     | ✅ Yes        | CHIRPS              | 5km resolution        |
| **Ghost Farmer Detection** | ✅ Yes        | WorldPop            | 100m resolution       |
| **Historical Analysis**    | ✅ Yes        | Sentinel-2 archive  | 2015+                 |
| **Flood Detection**        | ✅ Yes        | Sentinel-1 SAR      | All-weather           |
| **Drought Detection**      | ✅ Yes        | CHIRPS              | Historical comparison |
| **Cropland Validation**    | ✅ Yes        | Dynamic World       | Near real-time        |
| **Evidence Generation**    | ✅ Yes        | `getThumbURL()`     | PNG/JPEG export       |

### 14.2 Final Verdict

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│   ✅ ALL 7 FRAUD INDICATORS ARE FULLY IMPLEMENTABLE IN GEE          │
│                                                                      │
│   ✅ All required data sources are available                        │
│   ✅ All algorithms are supported                                   │
│   ✅ Evidence generation is possible                                │
│   ✅ Processing time target (<10 seconds) is achievable             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 15. VALIDATION & TESTING

### 15.1 Accuracy Targets

| Metric                       | Target       | Measurement Method                |
| ---------------------------- | ------------ | --------------------------------- |
| Fraud Detection F1 Score     | > 90%        | Confusion matrix on test set      |
| False Positive Rate          | < 10%        | Legitimate farms flagged as fraud |
| False Negative Rate          | < 5%         | Fraud cases missed                |
| Boundary Detection IoU       | > 80%        | Intersection over Union           |
| Crop Classification Accuracy | > 85%        | Against ground truth              |
| Processing Time              | < 10 seconds | Per farm verification             |

### 15.2 Test Dataset Requirements

| Category           | Minimum Quantity | Purpose                    |
| ------------------ | ---------------- | -------------------------- |
| Ground Truth Farms | 200+             | Accuracy validation        |
| Known Fraud Cases  | 50+              | Fraud detection validation |
| Various Crop Types | 100+             | Classification testing     |
| Different Regions  | 5+ countries     | Geographic robustness      |
| Edge Cases         | 50+              | Boundary condition testing |

### 15.3 Validation Process

```
VALIDATION WORKFLOW:

1. COLLECT GROUND TRUTH DATA
   ├── Partner with agricultural organizations
   ├── Conduct field surveys
   ├── Collect GPS boundaries of known farms
   └── Document known fraud cases

2. PREPARE TEST DATASET
   ├── Split into training (70%) and test (30%)
   ├── Ensure geographic diversity
   └── Include edge cases

3. RUN SYSTEM ON TEST DATA
   ├── Process all test farms
   ├── Record predictions
   └── Measure processing times

4. CALCULATE METRICS
   ├── Precision = TP / (TP + FP)
   ├── Recall = TP / (TP + FN)
   ├── F1 = 2 × (Precision × Recall) / (Precision + Recall)
   └── IoU = Intersection / Union

5. ANALYZE FAILURES
   ├── Identify patterns in false positives
   ├── Identify patterns in false negatives
   └── Adjust thresholds if needed

6. DOCUMENT RESULTS
   ├── Create accuracy report
   ├── Document limitations
   └── Provide recommendations
```

---

## APPENDIX: QUICK REFERENCE

### Indicator Scoring Summary

| Indicator              | Max Points | Thresholds                                 |
| ---------------------- | ---------- | ------------------------------------------ |
| Size Discrepancy       | 30         | 0-15%: 0, 15-30%: 10, 30-50%: 20, >50%: 30 |
| Crop Mismatch          | 30         | Match: 0, Similar: 15, Different: 30       |
| Weather Validation     | 20         | Adequate: 0, Mild deficit: 10, Severe: 20  |
| Ghost Farmer           | 20         | >10/km²: 0, 5-10: 10, <5: 20               |
| Historical Consistency | 15         | <0.15: 0, 0.15-0.30: 8, >0.30: 15          |
| Disaster Validation    | 10         | Confirmed: 0, Not confirmed: 10            |
| Cropland Signal        | 10         | High: 0, Medium: 5, Low: 10                |

### Risk Classification

| Score  | Risk Level | Action        |
| ------ | ---------- | ------------- |
| 0-39   | LOW        | APPROVE       |
| 40-69  | MEDIUM     | MANUAL_REVIEW |
| 70-100 | HIGH       | REJECT        |

---

_Document End - Fraud Detection Logic_
_Version 1.0 | December 2024_
_Meridian Sentinel - Agricultural Fraud Detection System_
