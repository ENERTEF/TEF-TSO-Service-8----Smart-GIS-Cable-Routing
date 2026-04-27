# Smart GIS Cable Routing

**Service:** Smart GIS Cable Routing  
**Document Type:** Technical Manual & Service Specification  
**Author:** Elektro Gorenjska d.d., Slovenian DSO  
**Version:** 1.0  
**Last Updated:** 15 Dec 2025  

---

# 1. Business Context & Definitions

Elektro Gorenjska infrastructure planners manually determine cable routes using QGIS tools. They balance criteria across layers including network development areas, connection capacities, networks, route sections with high/medium/low voltage poles and manholes, schematics for stations and separation points, parcels, and environment details such as company borders, local control areas, municipalities, addresses, streets, orthophoto grids, and LiDAR grids.

This process is time-consuming due to multiple factors including future infrastructure plans, terrain stability, and parcel approvals.

## Key Terms

| Term | Definition |
|---|---|
| GIS | Geographic Information System holding spatial layers for network, routes, parcels, schematics, and environment data |
| GIS layers | Separate map overlays representing different spatial data types (networks, parcels, streets, imagery, etc.) |
| Optimization algorithm | Computational method to evaluate multiple routes and minimize/balance defined costs |
| AI image recognition | Processing orthophoto and LiDAR data to identify terrain features such as instability or obstacles |
| Lowest-cost route | Path using existing infrastructure with zero additional construction cost |
| Parcel crossing | Path segment over private land parcels, minimized and centered for easier approvals |

## 1.1 Elektro Gorenjska (Network Owner) Context

This service targets Elektro Gorenjska’s full distribution network for planning:
- New cable routes  
- Rerouting existing cable sections  

Initial scope:
- Primarily MV and LV networks  
- Small HV sections  

### Objectives

- Reduce planning time by **≥50%**  
- Provide structured cost overview for candidate routes  

### Current baseline
- Max **3 projects/week/planner**

### Requirements

- Estimate deviation from lowest achievable cost  
- Use standardized internal unit costs (e.g., parcel crossing, excavation)  
- Evaluate trade-offs (longer route vs expensive segments)  

### Current manual workflow constraints

- Avoid water crossings  
- Prefer routing between parcels  
- Stay near roads but avoid excavation  
- Handle incomplete terrain data  
- Consider constraints not fully captured in GIS  

### Service capabilities

- Generate lowest-cost route  
- Provide alternative routes  
- Allow user editing (no real-time rerouting)  

### AI component

- Analyze orthophoto + LiDAR  
- Detect obstacles (e.g., boulders not in GIS)  

### Data & integration

- Standard GIS formats (shapefiles)  
- Imagery similar to PISO (10–50 cm orthophoto resolution)  
- Delivery as **QGIS plugin**  

---

# 2. Problem Statement

Manual routing is constrained by:

- Multi-layer geospatial optimization  
- Engineering, regulatory, economic limits  
- Limited planner capacity  

### Service objective

Automatically generate feasible route candidates between endpoints:
- Distribution cabinet → transformer station  
- Network point → network point  

### Optimization must include

- Construction accessibility  
- Land use constraints  
- Terrain suitability  
- Geometric rules  
- Capacity matching  
- Overlap avoidance  

### Technical approach

- GIS-integrated optimization (A*, ACO, least-cost path)  
- Multi-objective optimization (cost vs length vs risk)  
- Pareto-optimal routes  

### AI component

- Detect obstacles (≥90% accuracy for major obstacles >1m)  
- Flag uncertain cases  

### Outputs

- Lowest-cost route (existing infrastructure priority)  
- Ranked alternative routes  

### Prototype scope

- Internal testing on real projects  
- Evaluation vs manual planning  

---

# 3. Data Description

Core data sources:

- Internal GIS system (shapefiles `.shp`)  
- Public/internal imagery (orthophoto, LiDAR)  

### Imagery standards

- DOF010 / DOF050 (10–50 cm resolution)  
- LiDAR grids  
- TTN5/TTN10 maps  
- DTK25/TK50  
- PK250 overview maps  

---

## 3.1 GIS Layers Dictionary

| Layer Group | Specific Layers | Description | Data Type | Example Use |
|---|---|---|---|---|
| Network | Network, cable widths | Existing cables and sections | Line/Polygon | Prioritize for lowest-cost |
| Routes | Route sections, HV/MV/LV poles, manholes | Infrastructure points | Point/Line | Extend routes |
| Schematics | Stations, separation points, substations, joints | Network diagram elements | Point/Line | Endpoint candidates |
| Environment | Borders, municipalities, streets, orthophoto, LiDAR | Base mapping and terrain | Polygon/Line/Raster | Alignment |
| Other | Parcels, development, capacity, HC3 | Land use and planning | Polygon/Point | Avoid/prioritize |

### Notes

- Some attributes may exist but are undefined  
- LV feeders may serve ~100 consumers (to confirm)  
- Validation via synthetic GIS scenarios + expert review  

---

# 4. Analytics, Scope & Update Frequency

## Execution

- On-demand route optimization per project  
- Uses current GIS snapshot  

## Temporal Scope

- Current GIS + imagery only  
- No historical time series  

## Update Frequency

- Triggered by planner inputs  
- Instant recalculation on parameter change  

## Constraints

- Route length: <10 km  
- Nodes: <100  

---

## Output Format

Each project returns **two routes**:

### 1. Lowest-cost route
- Minimum total cost  
- May be €0 (existing infrastructure)  

### 2. Optimal route
- Balanced cost/length/terrain  
- Adjustable weights:
  - Parcels (e.g., 40%)  
  - Length (30%)  
  - Terrain (30%)  

### Additional features

- Granularity slider (5–50 m merge tolerance)  
- Adaptive smoothing (urban vs rural)  
- Interactive editor:
  - Snap to layers  
  - Drag/split nodes  
  - Auto-update costs  

### Outputs

- QGIS layers  
- Editable geometries/shapefiles  

### Cost parameters

- Imported via CSV/Excel  
- Example: €2000 per parcel crossing  

### Optional (future)

- Color-coded routes  
- Cost visualization tables  

---

# 5. Evaluation Protocols & Metrics

Evaluation verifies:
- Faster route generation  
- Equal or lower cost vs manual  

(after user training)

---

## 5.1 Data Usage & Analytical Protocol

- 30 synthetic scenarios  
- 10 anonymized real projects  
- ~5 planners  

Requirements:
- Training period before evaluation  
- Full reproducibility (inputs, parameters, outputs)  
- No future data leakage  

---

## 5.2 Data Gaps and Exceptions

| Case | Handling |
|---|---|
| Missing GIS layers | Exclude from metrics, flag |
| No LiDAR | Use orthophoto fallback |
| Extreme complexity (>100 nodes) | Route to expert |
| No historical routes | Use synthetic + expert validation |

---

## 5.3 Service Evaluation Metrics & KPIs

| KPI | Formula/Description | Target |
|---|---|---|
| Time Savings % | (manual − tool) / manual × 100 | ≥50% |
| Cost Accuracy | Avg % difference vs manual | ≤10% underrun |
| Task Throughput | Projects/week/person | ≥6 |
| Expert Acceptance | % planners preferring tool | ≥80% |
| Route Feasibility | % routes usable without major edits | ≥90% |

---

# 6. Deliverables & Submissions

Provider must deliver reports + technical artefacts.

---

## 6.1 Deliverable Reports

### 1. Pre-Service Deliverable – Service Design Setup Report

Includes:
- Optimization algorithm  
- Cost model  
- Granularity logic  
- Data requirements (shapefiles, CSV)  
- System architecture (QGIS plugin)  
- Integration plan  
- Operational procedures  
- Pilot schedule  

---

### 2. Intermediate Deliverable – Interim Performance Report (~6 weeks)

Includes:
- Early testing results  
- Synthetic + staff trials  
- KPI evaluation  
- Data coverage  
- Issues and improvements  

---

### 3. Final Deliverable – Final Evaluation Report (~3 months)

Includes:
- Full results (30 synthetic + 10 real)  
- 5-planner evaluation  
- Metrics:
  - ≥30% time savings  
  - ≥80% acceptance  
- Insights  
- Improvements  
- Scaling recommendations  

---

## 6.2 Technical Specifications & Submissions

### Plugin Artefacts
- QGIS plugin package  
- Git repository  
- Dockerfile  
- Installation guide  

### Configuration & Versioning
- CSV cost templates  
- Model parameters  
- Versioning  
- Handover procedures  

### User Documentation
- Input/output guide  
- Editor usage  

### Training Materials
- Video recordings  
- Training scripts  

---
