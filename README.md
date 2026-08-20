# N-choe Water Quality Monitoring & Visualization

## Overview

N-choe is a **60 km drainage channel** that flows from Chandigarh into the Ghgar River. Over the years, the drainage channel has been affected by industrial discharge, untreated sewage, municipal waste, and other pollutants, creating serious environmental concerns for aquatic ecosystems, agriculture, and nearby communities.

This project is an **interactive geospatial water-quality monitoring and visualization system** that transforms historical government datasets into an easy-to-use dashboard for analyzing pollution across the N-choe basin.

The project currently consists of two components:

* **Web Dashboard** — Interactive browser-based visualization for maps, charts, filtering, suitability analysis, and rainfall trends.

> **Status:** Ongoing research and development project.

---

## Problem Statement

Government water-quality observations are available across different years and monitoring stations, but they are often distributed across separate CSV files with inconsistent formatting and naming conventions.

The objective of this project is to create a unified platform that enables users to:

* Analyze historical water-quality measurements.
* Identify pollution hotspots geographically.
* Compare water-quality parameters across locations and years.
* Visualize spatial pollution distribution using interpolation.
* Evaluate water suitability for different environmental uses.
* Understand rainfall trends alongside water-quality observations.

The dashboard is intended primarily as a **decision-support and environmental monitoring tool**.

---

# Features

## Interactive Geospatial Dashboard

* Maps **13 monitoring stations** across the N-choe basin.
* Displays station name, latitude, longitude, and recorded measurements.
* Supports interactive station selection.
* Allows switching between multiple basemaps:

  * OpenStreetMap
  * Minimal
  * Satellite

## Water Quality Analysis

Users can explore parameters including:

* BOD
* COD
* pH
* Dissolved Oxygen
* TSS
* Conductivity
* TDS
* Coliform
* Additional available government measurements

The dashboard supports:

* Parameter filtering
* Year filtering
* Month filtering
* Station comparison
* Trend visualization

## Water Suitability Screening

Based on available environmental standards, the dashboard provides screening for:

* Drinking water
* Agriculture
* Bathing
* Aquatic life

> These results are intended for visualization and decision-support purposes based on available thresholds and should not replace official environmental certification.

## Historical Rainfall Integration

The dashboard integrates historical precipitation using the **Open-Meteo Archive API**.

Rainfall helps provide environmental context because precipitation can influence:

* Runoff
* Sewage overflow
* Pollutant transport
* Dilution
* Turbidity

## Spatial Pollution Visualization

The project uses **Ordinary Kriging** to estimate water-quality values between monitoring stations and generate a continuous spatial visualization.

A **river mask** generated from QGIS river segments ensures that interpolated values are displayed only within the mapped N-choe river corridor.

---

# System Architecture

```text
Government CSV Datasets
        │
        ▼
Data Cleaning & Preprocessing
        │
        ├───────────────┐
        ▼               ▼
dashboard-data.js   interpolation-data.json
                        │
                        ▼
                 River Mask (QGIS)
                        │
                        ▼
               river-mask-data.json
                        │
                        ▼
                Local Node.js Server
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
      Open-Meteo API        Browser Dashboard
                                  │
                                  ▼
                   Maps • Charts • Filters
                   Suitability • Rainfall
```

---

# Data Pipeline

## 1. Dataset Collection

Historical water-quality datasets are collected from multiple government and environmental sources.

Supported source years include:

* 2018
* 2023
* 2024
* 2025

The project automatically reads all CSV files from the `Datasets/` directory.

## 2. Data Preprocessing

Before visualization, all datasets are standardized into one common structure.

Cleaning includes:

* Normalizing parameter names
* Standardizing location names
* Cleaning text and encoding issues
* Converting month names into numeric values
* Creating sortable date fields
* Converting measurements into numeric values
* Preserving missing values as `null`
* Validating latitude and longitude
* Grouping equivalent monitoring stations

The result is a browser-ready dataset:

```text
dashboard-data.js
```

---

# Spatial Interpolation

Only 13 monitoring stations contain real measurements, while the river extends for approximately 60 km.

To estimate pollution between sampling locations, the project uses **Ordinary Kriging**.

### Workflow

```text
Monitoring Stations
(latitude, longitude, parameter)
            │
            ▼
Group by Parameter & Year
            │
            ▼
Average repeated station samples
            │
            ▼
Create 192 × 192 Grid
            │
            ▼
Ordinary Kriging
            │
            ▼
Interpolation Grid
            │
            ▼
interpolation-data.json
```

### Why Ordinary Kriging?

Ordinary Kriging is suitable for environmental spatial data because nearby observations generally exhibit greater similarity than distant observations.

The implementation:

* Uses nearby monitoring stations.
* Estimates values for unknown locations.
* Produces a continuous pollution surface.
* Uses IDW as a fallback if the Kriging matrix cannot be solved.

> The interpolation represents an estimated spatial distribution, **not confirmed measurements**.

---

# River Mask Generation

Interpolation alone would generate values across the entire map, including roads, buildings, and surrounding land.

To ensure pollution is displayed only inside the drainage channel, the project uses a **river mask**.

### Workflow

```text
QGIS Segmented River
        │
        ▼
GeoPackage (.gpkg)
Layer: segments
        │
        ▼
Python GeoPandas Script
        │
        ▼
192 × 192 Grid Intersection
        │
        ▼
river-mask-data.json
```

Each grid cell contains:

* `0` → Outside river corridor
* Positive value → Inside river corridor

The river mask acts as a **spatial filter**, allowing interpolated pollution values to appear only along the actual N-choe drainage network.

---

# Dashboard Functionality

When a user clicks a monitoring station, they can view:

* Station name
* Latitude & Longitude
* Available water-quality parameters
* Historical measurements
* Trend charts
* Rainfall trends
* Water suitability screening

The dashboard updates dynamically whenever users change:

* Station
* Parameter
* Year
* Month

---

# Technology Stack

| Area                  | Technology                          |
| --------------------- | ----------------------------------- |
| Frontend              | HTML5                               |
| Styling               | CSS3                                |
| Interactivity         | Vanilla JavaScript                  |
| Maps                  | Leaflet 1.9.4                       |
| Charts                | Chart.js 4.4.3                      |
| Local Server          | Node.js                             |
| Data Processing       | Node.js (`fs`, `path`, `http`)      |
| Weather               | Open-Meteo Archive API              |
| Geospatial Processing | Python                              |
| GIS                   | QGIS                                |
| Spatial Libraries     | GeoPandas, Shapely, PyProj, Pyogrio |
| Data Analysis         | Pandas, NumPy                       |
| Python Visualization  | Matplotlib, Folium                  |
| Prototype Dashboard   | Dash + Dash Bootstrap Components    |

---

# Project Structure

```text
N-choe/
│
├── Datasets/
│   └── Water quality CSV files
│
├── index.html
├── styles.css
├── app.js
│
├── build-dashboard-data.js
├── build-interpolation-data.js
├── serve-dashboard.js
│
├── build-river-mask.py
│
├── dashboard-data.js
├── interpolation-data.json
├── river-mask-data.json
│
└── requirements.txt
```

---

# Running the Dashboard

## Prerequisites

* Node.js 18 or above
* Python 3.11+ (for geospatial utilities)

## Start the Web Dashboard

```bash
node serve-dashboard.js
```

Open the dashboard in your browser:

```text
http://127.0.0.1:4173
```

The server automatically:

* Serves dashboard files
* Rebuilds processed datasets
* Watches CSV changes
* Provides the weather API proxy

---

# Rebuild Data Only

If you only want to regenerate processed data:

```bash
node build-dashboard-data.js
```

Generate interpolation:

```bash
node build-interpolation-data.js
```

---

---

# Water Quality Parameters

| Parameter        | Significance                                           |
| ---------------- | ------------------------------------------------------ |
| BOD              | Indicates organic pollution and oxygen demand in water |
| COD              | Measures chemical pollution load                       |
| pH               | Indicates acidity or alkalinity of water               |
| Dissolved Oxygen | Essential for aquatic life survival                    |
| TSS              | Measures suspended particles affecting water clarity   |
| Conductivity     | Indicates dissolved salts and ionic contamination      |
| TDS              | Represents dissolved solids in water                   |
| Coliform         | Indicates possible microbial contamination             |

---

# Target Users

### Government Agencies

The primary audience includes environmental monitoring authorities who need to:

* Identify pollution hotspots
* Compare monitoring stations
* Study long-term pollution trends
* Prioritize inspection and remediation efforts

### Researchers

Researchers can use the system to:

* Analyze environmental datasets
* Study parameter relationships
* Explore spatial pollution patterns
* Reuse the OpenGeoStreams toolkit

### Citizens

The dashboard also promotes public awareness by making river-health information easier to understand.

---


