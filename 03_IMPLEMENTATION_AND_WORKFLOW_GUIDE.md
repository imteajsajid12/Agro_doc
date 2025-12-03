# MERIDIAN SENTINEL - Implementation & Workflow Guide

## Document Version: 1.0

## Date: December 3, 2024

## Status: Technical Implementation Guide

---

## TABLE OF CONTENTS

1. [System Architecture](#1-system-architecture)
2. [Implementation Phases](#2-implementation-phases)
3. [Detailed Workflow Process](#3-detailed-workflow-process)
4. [Module Implementation Guide](#4-module-implementation-guide)
5. [Data Pipeline Architecture](#5-data-pipeline-architecture)
6. [API Design](#6-api-design)
7. [Deployment Strategy](#7-deployment-strategy)
8. [Testing Strategy](#8-testing-strategy)

---

## 1. SYSTEM ARCHITECTURE

### 1.1 High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           MERIDIAN SENTINEL                               │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐       │
│  │   CLIENT LAYER  │    │  BACKEND LAYER  │    │   GEE LAYER     │       │
│  │                 │    │                 │    │                 │       │
│  │  • Web Dashboard│───▶│  • API Server   │───▶│  • Processing   │       │
│  │  • Mobile App   │    │  • Auth/Queue   │    │  • Analysis     │       │
│  │  • API Access   │    │  • Database     │    │  • Indicators   │       │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘       │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Technology Stack Recommendation

| Layer               | Technology                          | Rationale                 |
| ------------------- | ----------------------------------- | ------------------------- |
| **Frontend**        | React + Next.js                     | Modern, fast, SSR support |
| **Backend API**     | Node.js + Express OR Python FastAPI | GEE SDK support           |
| **Database**        | PostgreSQL + PostGIS                | Geospatial support        |
| **Cache**           | Redis                               | Queue management, caching |
| **GEE Integration** | GEE JavaScript/Python API           | Core processing           |
| **Maps**            | Leaflet / Google Maps               | Visualization             |
| **Authentication**  | Auth0 / Firebase Auth               | Secure access             |

### 1.3 Component Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        FRONTEND (Next.js)                            │
├─────────────────────────────────────────────────────────────────────┤
│  Dashboard  │  Farm Map  │  Reports  │  Batch Upload  │  Settings   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         API GATEWAY                                  │
├─────────────────────────────────────────────────────────────────────┤
│  Rate Limiting  │  Authentication  │  Request Validation            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      BACKEND SERVICES                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │ Verification     │  │ Batch Processing │  │ Report Generator │   │
│  │ Service          │  │ Service          │  │ Service          │   │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘   │
│                                                                      │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │ Monitoring       │  │ Alert Service    │  │ Analytics        │   │
│  │ Service          │  │                  │  │ Service          │   │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     GEE PROCESSING ENGINE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐    │
│  │  Boundary   │ │    Crop     │ │   Health    │ │    Fraud    │    │
│  │  Detection  │ │  Classify   │ │  Monitor    │ │  Scoring    │    │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘    │
│                                                                      │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐    │
│  │  Weather    │ │ Population  │ │  Historical │ │  Evidence   │    │
│  │  Validation │ │   Check     │ │   Analysis  │ │  Generator  │    │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       DATA SOURCES (GEE)                             │
├─────────────────────────────────────────────────────────────────────┤
│  Sentinel-2 │ Sentinel-1 │ CHIRPS │ WorldPop │ Dynamic World │ GSW  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. IMPLEMENTATION PHASES

### 2.1 Phase Overview

| Phase                               | Duration  | Deliverables                            | Dependencies |
| ----------------------------------- | --------- | --------------------------------------- | ------------ |
| **Phase 1: Foundation**             | 3-4 weeks | Core infrastructure, basic GEE setup    | None         |
| **Phase 2: Core Modules**           | 4-6 weeks | Boundary detection, crop classification | Phase 1      |
| **Phase 3: Fraud Detection**        | 4-5 weeks | All 7 fraud indicators                  | Phase 2      |
| **Phase 4: Integration**            | 3-4 weeks | API, dashboard, reporting               | Phase 3      |
| **Phase 5: Testing & Optimization** | 2-3 weeks | Performance tuning, validation          | Phase 4      |

### 2.2 Phase 1: Foundation (Weeks 1-4)

**Objectives:**

- Set up GEE development environment
- Establish backend infrastructure
- Create database schema
- Implement authentication

**Tasks:**

```
Week 1-2:
├── Set up GEE project and service account
├── Initialize backend (Node.js/Python)
├── Design database schema
├── Set up PostgreSQL with PostGIS
└── Configure Redis for caching/queuing

Week 3-4:
├── Implement authentication (API keys, OAuth)
├── Create basic API endpoints
├── Set up CI/CD pipeline
├── Create development/staging environments
└── Write initial GEE connection code
```

**Deliverables:**

- [ ] GEE service account configured
- [ ] Backend API skeleton running
- [ ] Database with required tables
- [ ] Basic authentication working
- [ ] CI/CD pipeline configured

### 2.3 Phase 2: Core Modules (Weeks 5-10)

**Objectives:**

- Implement field boundary detection
- Implement crop classification
- Implement health monitoring
- Implement area calculation

**Tasks:**

```
Week 5-6: Boundary Detection
├── Implement SNIC segmentation algorithm
├── Implement Dynamic World fallback
├── Create boundary extraction logic
├── Implement area calculation
└── Write unit tests

Week 7-8: Crop Classification
├── Implement NDVI/EVI calculations
├── Create rules-based classification
├── Implement confidence scoring
├── Test against sample farms
└── Document thresholds

Week 9-10: Health Monitoring
├── Implement health score calculation
├── Create stress detection algorithms
├── Implement time-series analysis
├── Create anomaly detection
└── Integration testing
```

### 2.4 Phase 3: Fraud Detection (Weeks 11-15)

**Objectives:**

- Implement all 7 fraud indicators
- Create scoring algorithm
- Implement evidence package generation

**Tasks:**

```
Week 11-12: Core Indicators
├── Size Discrepancy (Indicator 1)
├── Crop Type Mismatch (Indicator 2)
├── Weather Validation (Indicator 3)
└── Unit tests for each

Week 13-14: Additional Indicators
├── Ghost Farmer Detection (Indicator 4)
├── Historical Consistency (Indicator 5)
├── Disaster Claim Validation (Indicator 6)
├── Cropland Signal (Indicator 7)
└── Infrastructure Presence (Indicator 8)

Week 15: Scoring & Integration
├── Implement total score calculation
├── Implement risk classification
├── Create recommendation engine
├── Build evidence package generator
└── Integration testing
```

### 2.5 Phase 4: Integration (Weeks 16-19)

**Objectives:**

- Build API layer
- Create dashboard
- Implement batch processing
- Build reporting system

**Tasks:**

```
Week 16-17: API Development
├── Design REST API endpoints
├── Implement farm verification endpoint
├── Implement batch upload endpoint
├── Implement monitoring endpoints
└── API documentation (OpenAPI/Swagger)

Week 18-19: Dashboard & Reports
├── Build farm verification UI
├── Create map visualization
├── Implement batch upload interface
├── Build report generation
└── Create export functionality (CSV, PDF)
```

### 2.6 Phase 5: Testing & Optimization (Weeks 20-22)

**Objectives:**

- Validate accuracy targets
- Optimize performance
- Load testing
- Documentation

---

## 3. DETAILED WORKFLOW PROCESS

### 3.1 Single Farm Verification Workflow

```
┌──────────────────────────────────────────────────────────────────────┐
│                    SINGLE FARM VERIFICATION WORKFLOW                  │
└──────────────────────────────────────────────────────────────────────┘

STEP 1: INPUT RECEPTION
┌─────────────────────────────────────────────────────────────────────┐
│  User Input:                                                         │
│  ├── Farmer ID: "FRM-12345"                                         │
│  ├── GPS Coordinates: [-1.2921, 36.8219] (lat, lon)                 │
│  ├── Claimed Area: 2.5 hectares                                     │
│  ├── Claimed Crop: "maize"                                          │
│  ├── Planting Date: "2024-03-15"                                    │
│  └── Disaster Claim: null (or "flood", "drought")                   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 2: DATA RETRIEVAL FROM GEE
┌─────────────────────────────────────────────────────────────────────┐
│  Parallel Data Fetch:                                                │
│  ├── [1] Sentinel-2 imagery (last 30 days, <20% cloud)              │
│  ├── [2] Dynamic World land cover                                    │
│  ├── [3] CHIRPS rainfall (6 months)                                  │
│  ├── [4] WorldPop population density                                 │
│  ├── [5] Sentinel-1 SAR (if flood claim)                            │
│  └── [6] Historical imagery (5 years ago)                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 3: BOUNDARY DETECTION
┌─────────────────────────────────────────────────────────────────────┐
│  Process:                                                            │
│  ├── Apply SNIC clustering to Sentinel-2                            │
│  ├── Extract segments containing farm point                         │
│  ├── Validate with Dynamic World cropland layer                     │
│  ├── Calculate detected boundary polygon                            │
│  └── Calculate detected area in hectares                            │
│                                                                      │
│  Output:                                                             │
│  ├── detectedBoundary: GeoJSON polygon                              │
│  ├── detectedArea: 2.1 hectares                                     │
│  └── boundaryConfidence: 0.85                                       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 4: CROP CLASSIFICATION
┌─────────────────────────────────────────────────────────────────────┐
│  Process:                                                            │
│  ├── Calculate NDVI, EVI for detected area                          │
│  ├── Apply crop-specific spectral rules                             │
│  ├── Compare to claimed crop type                                    │
│  └── Calculate classification confidence                            │
│                                                                      │
│  Output:                                                             │
│  ├── detectedCrop: "maize"                                          │
│  ├── cropMatch: true                                                │
│  └── classificationConfidence: 0.78                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 5: HEALTH ASSESSMENT
┌─────────────────────────────────────────────────────────────────────┐
│  Process:                                                            │
│  ├── Calculate current NDVI/EVI                                     │
│  ├── Compare to expected values for crop stage                      │
│  ├── Check for stress indicators                                    │
│  └── Calculate health score (0-100)                                 │
│                                                                      │
│  Output:                                                             │
│  ├── healthScore: 72                                                │
│  ├── healthStatus: "stressed"                                       │
│  └── stressType: "water_stress"                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 6: FRAUD INDICATOR CALCULATION
┌─────────────────────────────────────────────────────────────────────┐
│  Calculate Each Indicator:                                           │
│                                                                      │
│  [1] Size Discrepancy:                                              │
│      Claimed: 2.5ha, Detected: 2.1ha                                │
│      Difference: 16% → Score: 5 points                              │
│                                                                      │
│  [2] Crop Type Mismatch:                                            │
│      Claimed: maize, Detected: maize                                │
│      Match: Yes → Score: 0 points                                   │
│                                                                      │
│  [3] Weather Validation:                                            │
│      Required: 450mm, Actual: 520mm                                 │
│      Adequate: Yes → Score: 0 points                                │
│                                                                      │
│  [4] Ghost Farmer Detection:                                        │
│      Population density: 25 people/ha                               │
│      Populated: Yes → Score: 0 points                               │
│                                                                      │
│  [5] Historical Consistency:                                        │
│      NDVI change: 0.12                                              │
│      Stable: Yes → Score: 0 points                                  │
│                                                                      │
│  [6] Disaster Claim Validation:                                     │
│      No claim → Score: 0 points                                     │
│                                                                      │
│  [7] Cropland Signal:                                               │
│      Cropland probability: 85%                                      │
│      NDVI: 0.58                                                     │
│      Valid: Yes → Score: 0 points                                   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 7: TOTAL SCORE CALCULATION
┌─────────────────────────────────────────────────────────────────────┐
│  Total Raw Score:                                                    │
│  ├── Size Discrepancy:      5 points                                │
│  ├── Crop Mismatch:         0 points                                │
│  ├── Weather:               0 points                                │
│  ├── Ghost Farmer:          0 points                                │
│  ├── Historical:            0 points                                │
│  ├── Disaster:              0 points                                │
│  └── Cropland Signal:       0 points                                │
│  ─────────────────────────────────────                              │
│  TOTAL: 5 / 135 points                                              │
│                                                                      │
│  Scaled Score: (5/135) × 100 = 3.7                                  │
│                                                                      │
│  Risk Level: LOW (0-39)                                             │
│  Recommendation: APPROVE                                            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 8: EVIDENCE PACKAGE GENERATION
┌─────────────────────────────────────────────────────────────────────┐
│  Generate Evidence Package:                                          │
│  ├── Satellite image thumbnail                                      │
│  ├── Detected boundary overlay                                      │
│  ├── NDVI visualization                                             │
│  ├── Rainfall chart                                                 │
│  ├── Historical comparison                                          │
│  ├── Individual indicator breakdown                                 │
│  └── Human-readable summary                                         │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
STEP 9: OUTPUT
┌─────────────────────────────────────────────────────────────────────┐
│  Final Response:                                                     │
│  {                                                                   │
│    "farmerId": "FRM-12345",                                         │
│    "verificationId": "VER-2024-001234",                             │
│    "timestamp": "2024-12-03T10:30:00Z",                             │
│    "fraudScore": 3.7,                                               │
│    "riskLevel": "LOW",                                              │
│    "recommendation": "APPROVE",                                      │
│    "detectedArea": 2.1,                                             │
│    "claimedArea": 2.5,                                              │
│    "detectedCrop": "maize",                                         │
│    "healthScore": 72,                                               │
│    "indicators": { ... },                                           │
│    "evidence": { ... },                                             │
│    "processingTime": "8.2 seconds"                                  │
│  }                                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 Batch Processing Workflow

```
┌──────────────────────────────────────────────────────────────────────┐
│                    BATCH PROCESSING WORKFLOW                          │
└──────────────────────────────────────────────────────────────────────┘

INPUT: CSV file with 10,000+ farms
├── farmer_id, lat, lon, claimed_area, claimed_crop, planting_date

STEP 1: FILE VALIDATION
├── Validate CSV format
├── Check required columns
├── Validate coordinates
└── Report validation errors

STEP 2: CHUNKING
├── Divide into batches of 100 farms
├── Create job queue in Redis
└── Initialize progress tracking

STEP 3: PARALLEL PROCESSING
┌─────────────────────────────────────────────────────────────────────┐
│  Worker 1: Batch 1-10                                                │
│  Worker 2: Batch 11-20                                               │
│  Worker 3: Batch 21-30                                               │
│  ...                                                                 │
│  Worker N: Final batches                                             │
└─────────────────────────────────────────────────────────────────────┘

STEP 4: RESULT AGGREGATION
├── Collect all results
├── Calculate statistics
├── Generate risk distribution
└── Create summary report

STEP 5: OUTPUT GENERATION
├── Full results CSV
├── Summary report PDF
├── Risk distribution chart
└── Flagged farms list
```

---

## 4. MODULE IMPLEMENTATION GUIDE

### 4.1 Boundary Detection Module

**File Structure:**

```
/gee-modules/
├── boundary/
│   ├── snic-segmentation.js
│   ├── dynamic-world-fallback.js
│   ├── boundary-extraction.js
│   ├── area-calculation.js
│   └── index.js
```

**Core Implementation:**

```javascript
// boundary/snic-segmentation.js

/**
 * Detect farm boundary using SNIC segmentation
 * @param {ee.Geometry.Point} point - Farm center point
 * @param {number} radius - Search radius in meters (default: 300)
 * @returns {Object} Boundary detection result
 */
function detectBoundaryWithSNIC(point, radius) {
  radius = radius || 300;
  var region = point.buffer(radius);

  // Get recent Sentinel-2 imagery
  var s2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(region)
    .filterDate(ee.Date(Date.now()).advance(-30, "day"), ee.Date(Date.now()))
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median()
    .clip(region);

  // Apply SNIC segmentation
  var seeds = ee.Algorithms.Image.Segmentation.seedGrid(10);
  var snic = ee.Algorithms.Image.Segmentation.SNIC({
    image: s2.select(["B4", "B3", "B2", "B8"]),
    size: 10,
    compactness: 0,
    connectivity: 8,
    neighborhoodSize: 50,
    seeds: seeds,
  });

  // Get cluster containing the farm point
  var clusterAtPoint = snic.select("clusters").reduceRegion({
    reducer: ee.Reducer.first(),
    geometry: point,
    scale: 10,
  });

  var clusterId = ee.Number(clusterAtPoint.get("clusters"));

  // Create mask for the specific cluster
  var farmMask = snic.select("clusters").eq(clusterId);

  // Convert to vector
  var farmVector = farmMask.selfMask().reduceToVectors({
    geometry: region,
    scale: 10,
    geometryType: "polygon",
    maxPixels: 1e8,
  });

  // Get the farm boundary
  var boundary = farmVector.first().geometry();

  // Calculate area
  var areaHectares = boundary.area().divide(10000);

  return {
    boundary: boundary,
    area: areaHectares,
    method: "SNIC",
    confidence: calculateConfidence(s2, boundary),
  };
}

/**
 * Calculate boundary detection confidence
 */
function calculateConfidence(image, boundary) {
  // Calculate NDVI standard deviation within boundary
  var ndvi = image.normalizedDifference(["B8", "B4"]);
  var stats = ndvi.reduceRegion({
    reducer: ee.Reducer.stdDev(),
    geometry: boundary,
    scale: 10,
  });

  // Lower std dev = more homogeneous = higher confidence
  var stdDev = ee.Number(stats.get("nd"));
  var confidence = ee.Number(1).subtract(stdDev.min(1));

  return confidence;
}

exports.detectBoundaryWithSNIC = detectBoundaryWithSNIC;
```

### 4.2 Crop Classification Module

```javascript
// crop/classification.js

/**
 * Classify crop type using spectral indices
 * @param {ee.Geometry} farmBoundary - Detected farm boundary
 * @param {string} claimedCrop - Farmer's claimed crop type
 * @returns {Object} Classification result
 */
function classifyCrop(farmBoundary, claimedCrop) {
  // Get growing season imagery
  var s2 = ee
    .ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
    .filterBounds(farmBoundary)
    .filterDate("2024-06-01", "2024-09-30")
    .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 20))
    .median();

  // Calculate spectral indices
  var ndvi = s2.normalizedDifference(["B8", "B4"]).rename("NDVI");
  var evi = s2
    .expression("2.5 * ((NIR - RED) / (NIR + 6 * RED - 7.5 * BLUE + 1))", {
      NIR: s2.select("B8"),
      RED: s2.select("B4"),
      BLUE: s2.select("B2"),
    })
    .rename("EVI");

  // Get mean values within farm
  var stats = ndvi.addBands(evi).reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: farmBoundary,
    scale: 10,
  });

  var ndviVal = ee.Number(stats.get("NDVI"));
  var eviVal = ee.Number(stats.get("EVI"));

  // Rules-based classification
  var detectedCrop = classifyByRules(ndviVal, eviVal);

  // Compare with claimed crop
  var cropMatch = ee.String(detectedCrop).equals(ee.String(claimedCrop));

  return {
    detectedCrop: detectedCrop,
    claimedCrop: claimedCrop,
    cropMatch: cropMatch,
    ndvi: ndviVal,
    evi: eviVal,
    confidence: calculateCropConfidence(ndviVal, eviVal, detectedCrop),
  };
}

/**
 * Crop classification rules based on spectral characteristics
 */
var cropRules = {
  rice: { ndviMin: 0.3, ndviMax: 0.6, eviMin: 0.2, eviMax: 0.5 },
  maize: { ndviMin: 0.5, ndviMax: 0.8, eviMin: 0.4, eviMax: 0.7 },
  cassava: { ndviMin: 0.4, ndviMax: 0.7, eviMin: 0.3, eviMax: 0.6 },
  sorghum: { ndviMin: 0.4, ndviMax: 0.7, eviMin: 0.3, eviMax: 0.6 },
  beans: { ndviMin: 0.3, ndviMax: 0.6, eviMin: 0.2, eviMax: 0.5 },
  millet: { ndviMin: 0.3, ndviMax: 0.5, eviMin: 0.2, eviMax: 0.4 },
};
```

### 4.3 Fraud Scoring Module

```javascript
// fraud/scoring.js

/**
 * Calculate total fraud score from all indicators
 * @param {Object} indicators - All calculated indicator scores
 * @returns {Object} Final fraud assessment
 */
function calculateFraudScore(indicators) {
  // Sum all indicator scores
  var totalRawScore =
    indicators.sizeDiscrepancy + // Max 30
    indicators.cropMismatch + // Max 30
    indicators.weatherValidation + // Max 20
    indicators.ghostFarmer + // Max 20
    indicators.historicalConsistency + // Max 15
    indicators.disasterValidation + // Max 10
    indicators.croplandSignal; // Max 10

  // Maximum possible score
  var maxScore = 135;

  // Scale to 0-100
  var scaledScore = (totalRawScore / maxScore) * 100;

  // Determine risk level
  var riskLevel, recommendation;
  if (scaledScore >= 70) {
    riskLevel = "HIGH";
    recommendation = "REJECT";
  } else if (scaledScore >= 40) {
    riskLevel = "MEDIUM";
    recommendation = "MANUAL_REVIEW";
  } else {
    riskLevel = "LOW";
    recommendation = "APPROVE";
  }

  return {
    rawScore: totalRawScore,
    scaledScore: scaledScore,
    riskLevel: riskLevel,
    recommendation: recommendation,
    indicators: indicators,
  };
}
```

---

## 5. DATA PIPELINE ARCHITECTURE

### 5.1 Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA PIPELINE FLOW                            │
└─────────────────────────────────────────────────────────────────────┘

     ┌──────────────┐
     │  USER INPUT  │
     │  (API/CSV)   │
     └──────┬───────┘
            │
            ▼
┌──────────────────────┐
│   INPUT VALIDATION   │
│  ├─ Coordinate check │
│  ├─ Schema validation│
│  └─ Duplicate check  │
└──────────┬───────────┘
            │
            ▼
┌──────────────────────┐
│    JOB QUEUE        │
│    (Redis)          │
└──────────┬───────────┘
            │
            ▼
┌──────────────────────────────────────────────────────────────────┐
│                    GEE PROCESSING PIPELINE                        │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  STAGE 1: DATA RETRIEVAL (Parallel)                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │Sentinel-2│ │ Dynamic  │ │  CHIRPS  │ │ WorldPop │            │
│  │          │ │  World   │ │          │ │          │            │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘            │
│       └────────────┴────────────┴────────────┘                   │
│                            │                                      │
│                            ▼                                      │
│  STAGE 2: ANALYSIS                                               │
│  ┌──────────────────────────────────────────────────┐            │
│  │  Boundary Detection → Crop Classification →      │            │
│  │  Health Assessment → Fraud Indicators            │            │
│  └──────────────────────────────────────────────────┘            │
│                            │                                      │
│                            ▼                                      │
│  STAGE 3: SCORING & EVIDENCE                                     │
│  ┌──────────────────────────────────────────────────┐            │
│  │  Calculate Score → Generate Evidence →           │            │
│  │  Create Report                                   │            │
│  └──────────────────────────────────────────────────┘            │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
            │
            ▼
┌──────────────────────┐
│   RESULT STORAGE     │
│   (PostgreSQL)       │
└──────────┬───────────┘
            │
            ▼
┌──────────────────────┐
│   API RESPONSE /     │
│   EXPORT (CSV/PDF)   │
└──────────────────────┘
```

---

## 6. API DESIGN

### 6.1 REST API Endpoints

| Method | Endpoint                           | Description                 |
| ------ | ---------------------------------- | --------------------------- |
| POST   | `/api/v1/verify`                   | Verify single farm          |
| POST   | `/api/v1/verify/batch`             | Upload batch CSV            |
| GET    | `/api/v1/verify/{id}`              | Get verification result     |
| GET    | `/api/v1/jobs/{jobId}`             | Get batch job status        |
| GET    | `/api/v1/farms/{farmerId}/history` | Get farmer history          |
| GET    | `/api/v1/reports/summary`          | Get summary report          |
| POST   | `/api/v1/monitor/start`            | Start continuous monitoring |

### 6.2 Request/Response Examples

**Single Farm Verification Request:**

```json
POST /api/v1/verify
{
  "farmerId": "FRM-12345",
  "coordinates": {
    "latitude": -1.2921,
    "longitude": 36.8219
  },
  "claimedArea": 2.5,
  "claimedCrop": "maize",
  "plantingDate": "2024-03-15",
  "disasterClaim": null
}
```

**Verification Response:**

```json
{
  "status": "success",
  "verificationId": "VER-2024-001234",
  "timestamp": "2024-12-03T10:30:00Z",
  "result": {
    "fraudScore": 12.5,
    "riskLevel": "LOW",
    "recommendation": "APPROVE",
    "detectedArea": 2.3,
    "areaDiscrepancy": "8%",
    "detectedCrop": "maize",
    "cropMatch": true,
    "healthScore": 78,
    "healthStatus": "healthy",
    "indicators": {
      "sizeDiscrepancy": { "score": 0, "detail": "Within 15% tolerance" },
      "cropMismatch": { "score": 0, "detail": "Crop types match" },
      "weatherValidation": { "score": 0, "detail": "Adequate rainfall: 520mm" },
      "ghostFarmer": { "score": 0, "detail": "Population: 45 per km²" },
      "historicalConsistency": {
        "score": 5,
        "detail": "Minor NDVI change: 0.18"
      },
      "disasterValidation": { "score": 0, "detail": "No claim" },
      "croplandSignal": { "score": 0, "detail": "Cropland probability: 92%" }
    },
    "evidence": {
      "satelliteImageUrl": "https://...",
      "boundaryMapUrl": "https://...",
      "ndviChartUrl": "https://...",
      "rainfallChartUrl": "https://..."
    }
  },
  "processingTime": "8.2s"
}
```

---

## 7. DEPLOYMENT STRATEGY

### 7.1 Infrastructure Requirements

| Component          | Specification               | Purpose              |
| ------------------ | --------------------------- | -------------------- |
| Application Server | 4 vCPU, 16GB RAM            | API handling         |
| Worker Servers     | 2-4 × 2 vCPU, 8GB RAM       | Batch processing     |
| Database Server    | 4 vCPU, 16GB RAM, 500GB SSD | PostgreSQL           |
| Redis Server       | 2 vCPU, 4GB RAM             | Caching, queues      |
| Load Balancer      | Cloud-based                 | Traffic distribution |

### 7.2 Recommended Cloud Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CLOUD DEPLOYMENT ARCHITECTURE                     │
└─────────────────────────────────────────────────────────────────────┘

                        ┌─────────────────┐
                        │   CloudFlare    │
                        │   (CDN + WAF)   │
                        └────────┬────────┘
                                 │
                                 ▼
                        ┌─────────────────┐
                        │ Load Balancer   │
                        │ (NGINX/HAProxy) │
                        └────────┬────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            │                    │                    │
            ▼                    ▼                    ▼
    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
    │   API Server  │   │   API Server  │   │   API Server  │
    │   (Node.js)   │   │   (Node.js)   │   │   (Node.js)   │
    └───────┬───────┘   └───────┬───────┘   └───────┬───────┘
            │                   │                   │
            └───────────────────┼───────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│    Redis      │      │  PostgreSQL   │      │    GEE API    │
│   (Cache)     │      │   (Database)  │      │   (External)  │
└───────────────┘      └───────────────┘      └───────────────┘
```

---

## 8. TESTING STRATEGY

### 8.1 Test Categories

| Test Type         | Scope                | Tools           |
| ----------------- | -------------------- | --------------- |
| Unit Tests        | Individual functions | Jest/Mocha      |
| Integration Tests | API endpoints        | Supertest       |
| GEE Tests         | Earth Engine code    | GEE Code Editor |
| Accuracy Tests    | Model validation     | Python/Jupyter  |
| Load Tests        | Performance          | k6/Artillery    |
| E2E Tests         | Full workflow        | Cypress         |

### 8.2 Validation Dataset Requirements

| Category           | Quantity     | Source               |
| ------------------ | ------------ | -------------------- |
| Ground Truth Farms | 200+         | Field surveys        |
| Known Fraud Cases  | 50+          | Historical records   |
| Various Crop Types | 100+         | Distributed          |
| Different Regions  | 5+ countries | Geographic diversity |

### 8.3 Accuracy Validation Process

```
VALIDATION WORKFLOW:

1. PREPARE TEST DATASET
   ├── Collect ground truth farms with known boundaries
   ├── Collect known fraud cases
   └── Include variety of crops, sizes, regions

2. RUN SYSTEM ON TEST DATA
   ├── Process all test farms through system
   ├── Record all predictions and scores
   └── Measure processing times

3. CALCULATE METRICS
   ├── Boundary Detection: IoU score
   ├── Crop Classification: Confusion matrix
   ├── Fraud Detection: Precision, Recall, F1
   ├── Area Calculation: Mean Absolute Error
   └── Processing Time: Mean, P95

4. ANALYZE RESULTS
   ├── Identify failure patterns
   ├── Adjust thresholds if needed
   └── Document limitations

5. ITERATE
   ├── Fix issues
   ├── Re-validate
   └── Document final accuracy
```

---

_Document End - Implementation & Workflow Guide_
_Version 1.0 | December 2024_
