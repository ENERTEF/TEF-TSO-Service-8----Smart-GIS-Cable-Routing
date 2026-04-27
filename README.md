# Smart GIS Cable Routing

**Service:** Smart GIS Cable Routing  
**Document Type:** Technical Manual & Service Specification  
**Author:** Elektro Gorenjska d.d., Slovenian DSO  
**Version:** 1.0  
**Last Updated:** 15 Dec 2025  

---

# 1. Business Context & Definitions

Elektro Gorenjska infrastructure planners manually determine cable routes using QGIS tools. They balance criteria across layers including:

- network development areas  
- connection capacities  
- networks  
- route sections with high/medium/low voltage poles and manholes  
- schematics for stations and separation points  
- parcels  
- environment details like company borders, local control areas, municipalities, addresses, streets, orthophoto grids, and LiDAR grids  

This process is time-consuming due to multiple factors like:

- future infrastructure plans  
- terrain stability  
- parcel approvals  

## Key Terms

| Term | Definition |
|-----|-----------|
| GIS | Geographic Information System holding spatial layers for network, routes, parcels, schematics, and environment data |
| GIS layers | Separate map overlays where each represents one type of spatial information (networks, parcels, streets, imagery, etc.) |
| Optimization algorithm | Computational method evaluating routes and minimizing/balancing costs |
| AI image recognition | Processing orthophoto and LiDAR data to identify terrain features such as instability or obstacles |
| Lowest-cost route | Path using existing infrastructure with zero additional construction cost |
| Parcel crossing | Path segment over private land parcels, minimized and centered for easier approvals |

---

## 1.1 Elektro Gorenjska (network owner) Context

This service targets Elektro Gorenjska DSO’s full distribution network for:

- planning new cable routes  
- rerouting existing cable sections  

Scope:

- primarily medium-voltage (MV) and low-voltage (LV) networks  
- small sections of high-voltage (HV) network  

### Primary goals

- Reduce planning time by **at least 50%**
- Provide structured cost overview for candidate routes

### Current baseline

- Maximum of **3 projects per week per planner (5 working days)**

### Cost evaluation requirement

The service should estimate:

- how far each candidate route is from the **lowest achievable cost**
- using standardized unit costs from internal catalogues (e.g. parcel crossings, excavation, construction)
- comparing trade-offs such as:
  - longer detours vs expensive segments  

### Current manual workflow

Planners manually:

- combine multiple GIS layers  
- avoid water crossings  
- prefer routes between parcels  
- stay near roads but avoid excavation  
- consider public roads (mapped) vs private roads (not mapped)  
- account for terrain not fully captured in GIS  

### Expected system behavior

- propose an initial lowest-cost route  
- generate alternative routes  
- allow user editing  
- no real-time outage rerouting  

### AI component

Machine vision should:

- analyze orthophoto + LiDAR  
- detect obstacles (e.g. boulders missing in GIS layers)  

### Data & delivery

- GIS export formats (shapefiles)  
- imagery comparable to Slovenian GIS systems (e.g. PISO):
  - orthophoto 50 cm resolution  
  - 10 cm / 50 cm variants  

- preferred delivery:
  - **QGIS plugin**
  - outputs adapted for professional GIS workflows  

---

# 2. Problem Statement

Manual cable routing requires optimization across:

- geospatial constraints  
- engineering rules  
- economic considerations  
- regulatory limits  

### Current limitation

- max 3 projects/week per planner → bottleneck  

### Service objective

Automate generation of route candidates between endpoints:

- cabinet → transformer station  
- network point → network point  

### Optimization must include

- construction accessibility (prefer roadside verges)  
- land use constraints (minimize parcel crossings, avoid water)  
- terrain suitability (slopes, AI-detected obstacles)  
- geometric rules:
  - minimum bend radii  
  - radial feeder layout  
- capacity matching (cable sizing models)  
- overlap avoidance (feeder zones, multi-supplier houses)  

### Voltage-specific complexity

- LV: most constrained (dense obstacles)  
- MV/HV: thicker cable constraints  
- HV: additional geometric rules (larger turn radii)  

### Technical solution

- GIS-integrated optimization algorithms:
  - least-cost path  
  - A*  
  - Ant Colony Optimization (ACO)  

- multi-objective optimization (length vs cost vs risk)  
- Pareto-optimal route generation  

### AI component

- detect obstacles with ≥90% accuracy (e.g. boulders >1m)  
- flag uncertain detections  

### Output priorities

1. lowest-cost route (existing infrastructure first)  
2. ranked alternatives  

### Prototype

- tested on real Elektro Gorenjska projects  
- evaluated vs manual planning (time, cost, constraints)  

---

# 3. Data Description

Core data sources:

- internal GIS system (shapefiles `.shp`)  
- imagery layers (public + internal)  

### Imagery sources

- orthophoto (DOF010 / DOF050, 10–50 cm resolution)  
- LiDAR grids  
- topographic maps:
  - TTN5 / TTN10  
  - DTK25 / TK50  
  - PK250  

---

## 3.1 GIS Layers Dictionary

| Layer Group | Specific Layers | Description | Data Type | Example Use |
|------------|----------------|------------|-----------|------------|
| Network | Network, cable widths | Existing cables and sections | Line/Polygon | Prioritize for lowest-cost |
| Routes | Route sections, HV/MV/LV poles, manholes | Infrastructure points | Point/Line | Extend routes |
| Schematics | Station labels, separation points, HV/MV sections, substations, joints | Network diagram elements | Point/Line | Endpoint candidates |
| Environment | Company borders, control areas, municipalities, addresses, streets, orthophoto, LiDAR | Mapping and terrain | Polygon/Line/Raster | Alignment, terrain |
| Other | Parcels, development plans, capacity, HC3 | Land use and planning | Polygon/Point | Avoid/prioritize |

### Notes

- Attributes like parcel ownership or road type may exist but are undefined  
- LV feeders estimated ~100 consumers (to confirm)  
- Validation uses:
  - synthetic GIS scenarios  
  - expert review  

---

# 4. Analytics, Scope & Update Frequency

## Execution

- On-demand per project  
- Uses current GIS snapshot  

## Temporal Scope

- No historical data required  
- Includes future planning layers  

## Update Frequency

- Triggered by planner input  
- Instant recalculation  

## Constraints

- route length <10 km  
- <100 nodes  

---

## Output Format

Each project returns:

### 1. Lowest-cost route

- Minimum total cost  
- May be €0 (existing infrastructure) or e.g. €100k  

### 2. Optimal route

- Balanced cost/criteria  
- adjustable weights:
  - parcels (40%)  
  - length (30%)  
  - terrain (30%)  

### Additional features

- granularity slider:
  - merge tolerance 5–50m  
  - straightens path  
  - adaptive behavior:
    - finer urban  
    - coarser rural  

- interactive editor:
  - snapping  
  - dragging  
  - splitting nodes  
  - auto cost updates  

### Output format

- QGIS plugin layers  
- editable shapefiles  

### Cost input

- CSV / Excel import  
- example:
  - €2000 per parcel crossing  

### Optional (future)

- color-coded paths  
- cost visualization tables  

---

# 5. Evaluation Protocols & Metrics

Goal:

- faster route generation  
- equal or lower cost vs manual  

(after training period)

---

## 5.1 Data Usage & Analytical Protocol

- 30 synthetic scenarios  
- 10 anonymized real projects  
- 5 planners  

Requirements:

- training period before evaluation  
- reproducibility (inputs, parameters, outputs)  
- no future data leakage  

---

## 5.2 Data Gaps and Exceptions

| Case | Handling |
|-----|--------|
| Missing GIS layers | Exclude from metrics, flag |
| Missing LiDAR | fallback to orthophoto |
| Extreme cases (>100 nodes) | escalate to expert |
| No historical routes | use synthetic + expert validation |

---

## 5.3 Service Evaluation Metrics & KPIs

| KPI | Formula/Description | Target |
|----|-------------------|--------|
| Time Savings % | (manual time – tool time) / manual time × 100 | ≥50% |
| Cost Accuracy | Avg % difference vs manual quote | ≤10% underrun |
| Task Throughput | Projects/person/week | ≥6 |
| Expert Acceptance | % planners preferring tool | ≥80% |
| Route Feasibility | % routes usable without major edits | ≥90% |

---

# 6. Deliverables & Submissions

Provider delivers reports + technical artefacts.

---

## 6.1 Deliverable Reports

### 1. Pre-Service Deliverable – Service Design Setup Report

Includes:

- optimization algorithm  
- cost model  
- granularity logic  
- data requirements (shapefiles, CSV)  
- system architecture (QGIS plugin)  
- integration plan  
- operational procedures  
- pilot schedule  

---

### 2. Intermediate Deliverable – Interim Performance Report (~6 weeks)

Includes:

- early testing results  
- synthetic + staff trials  
- KPI performance  
- data coverage  
- issues and adaptations  

---

### 3. Final Deliverable – Final Evaluation Report (~3 months)

Includes:

- full results (30 synthetic + 10 real projects)  
- planner trial (5 planners)  
- metrics:
  - time savings ≥30%  
  - acceptance ≥80%  

- insights  
- improvements  
- scaling recommendations  

---

## 6.2 Technical Specifications & Submissions

### Plugin Artefacts

- QGIS plugin  
- Git repository  
- Dockerfile  
- installation guide  

### Configuration & Versioning

- CSV cost templates  
- model parameters  
- versioning  
- handover procedures  

### User Documentation

- input/output instructions  
- editor usage  

### Training Materials

- video recordings  
- onboarding scripts  

---
