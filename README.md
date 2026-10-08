# Shortest vs. Fastest Network Routing Analysis (QGIS & Graph Algorithms)

![QGIS](https://img.shields.io/badge/QGIS-3.34_LTR-588240?logo=qgis&logoColor=white)
![Analysis](https://img.shields.io/badge/Methodology-Dijkstra_Graph_Solver-orange)
![CRS](https://img.shields.io/badge/Projection-EPSG%3A3035%20(ETRS89)-003399)

## Project Overview
This repository presents a spatial routing analysis using QGIS Processing Network Analysis algorithms (Dijkstra's Shortest Path Solver) on OpenStreetMap vector topologies in [Brussels](http://googleusercontent.com/map_location_reference/0). 

The project compares **Geodesic Shortest Routes** (minimizing distance in meters) against **Time-Impedance Fastest Routes** (minimizing travel time based on legal speed profiles) across 5 strategic Origin-Destination (OD) pairs.

## Objectives
1. **Network Topology Validation:** Fix geometry disconnections, zero-length lines, and isolated nodes in vector road graphs using QGIS native topology toolsets.
2. **Impedance & Speed Attribute Modeling:** Calculate line segment travel duration (`cost_min`) using field expressions derived from speed attributes (`maxspeed` / OSM highway hierarchy).
3. **Graph Solving (Dijkstra Algorithm):** Compute optimal paths for 5 digitized OD pairs under two cost functions:
   * **Cost Metric 1:** Length minimization (`$length`).
   * **Cost Metric 2:** Time minimization (`$length / (speed_kmh * 1000 / 60)`).
4. **Comparative Analysis & Cartography:** Quantify detour ratios (trade-offs between extra distance vs. time saved) and present side-by-side comparative cartography.

---

## Speed Profile & Impedance Matrix

When explicit `maxspeed` attributes were missing in raw OSM data, standard European urban speed assumptions were applied based on functional classification:

| Functional Class (`highway`) | Default Speed Profile | Distance Cost Weight | Time Cost ($min / km$) |
| :--- | :---: | :---: | :---: |
| **Motorway / Trunk** | 100 km/h | High | 0.60 min/km |
| **Primary / Secondary** | 50 km/h | Medium | 1.20 min/km |
| **Tertiary** | 30 km/h | Medium | 2.00 min/km |
| **Residential / Local** | 30 km/h | Low | 2.00 min/km |
| **Living Street / Service** | 15 km/h | Excluded / Low | 4.00 min/km |

---

## Workflow Implementation

### Step 1: Attribute & Impedance Calculation
* **QGIS Manual Reference:** *Section 6.3.1 - Field Calculator*
* Added speed attribute field `speed_kmh`:
  ```sql
  CASE 
    WHEN "maxspeed" IS NOT NULL AND "maxspeed" > 0 THEN "maxspeed"
    WHEN "highway" IN ('motorway', 'trunk') THEN 100
    WHEN "highway" IN ('primary', 'secondary') THEN 50
    WHEN "highway" = 'tertiary' THEN 30
    ELSE 20
  END
