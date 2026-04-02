# InstaWarn

**Automated Impact-Based Multi-Hazard Early Warning Middleware for Bangladesh**

From forecast issuance to last-mile protective action: programmatic, reproducible, and built entirely on open data.

<div align="center">

[![Live Prototype](https://img.shields.io/badge/Live_Prototype-HuggingFace-FFD21E?style=for-the-badge&logo=huggingface&logoColor=black)](https://huggingface.co/spaces/jubayerahmad/InstaWarn)
[![Repository](https://img.shields.io/badge/Source_Code-GitHub-181717?style=for-the-badge&logo=github)](https://github.com/jubayer360/InstaWarn)
[![License](https://img.shields.io/badge/License-Open_Source-3DA639?style=for-the-badge)](#data-sources)
[![Pipeline](https://img.shields.io/badge/Pipeline_Runtime-76s-blue?style=for-the-badge)](#validation-cyclone-mocha-2023-hindcast)

</div>

---

<div align="center">
  <h2>🏆 Mapathon 2026 — 2nd Runner-Up</h2>
  <img src="https://lh3.googleusercontent.com/d/1Zp5rbvmQf0vqwZ0at85XrtHBdBPgQBYF" alt="Mapathon 2026 Prize" width="80%">
  <br><br>
  <b>Selected among the top 11 finalist teams from 29 competing universities. The only team from a business administration discipline among exclusively engineering and computer science finalists.</b>
</div>

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [System Overview](#system-overview)
- [System Architecture](#system-architecture)
- [Module Specifications](#module-specifications)
- [Validation: Cyclone Mocha 2023 Hindcast](#validation-cyclone-mocha-2023-hindcast)
- [The School-Shelter Paradox](#the-school-shelter-paradox)
- [Channel Routing for Isolated Communities](#channel-routing-for-isolated-communities)
- [Data Limitations and Operational Honesty](#data-limitations-and-operational-honesty)
- [Technical Stack](#technical-stack)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Data Sources](#data-sources)
- [Scope and Constraints](#scope-and-constraints)
- [Team InstaWarn](#team-instawarn)

---

## Problem Statement

Bangladesh's national early warning infrastructure (BMD, FFWC, Cyclone Preparedness Programme) is among the most mature in South Asia. InstaWarn does not replace any of it.

It addresses a specific, persistent, and documented gap: **the translation layer between a national-level forecast and a community-level protective action.**

This gap manifests across five structural failure points, each documented in post-disaster evaluations and humanitarian after-action reviews:

```mermaid
flowchart LR
    subgraph FAILURE["Structural Failure Points in Warning Dissemination"]
        direction TB
        F1["⏱ <b>Temporal Decay</b><br/>6-tier bureaucratic relay<br/>adds 14+ hours of latency"]
        F2["🔤 <b>Semantic Mismatch</b><br/>Meteorological terminology<br/>without localized context"]
        F3["📡 <b>Channel Homogeneity</b><br/>Uniform channel mix ignores<br/>connectivity heterogeneity"]
        F4["🔍 <b>Impact Opacity</b><br/>Hazard description without<br/>consequence quantification"]
        F5["🏫 <b>No Child Protocol</b><br/>No automated school-level<br/>anticipatory action logic"]
    end

    F1 --> F2 --> F3 --> F4 --> F5

    style FAILURE fill:#1a1a2e,stroke:#e94560,stroke-width:2px,color:#eee
    style F1 fill:#16213e,stroke:#e94560,color:#eee
    style F2 fill:#16213e,stroke:#e94560,color:#eee
    style F3 fill:#16213e,stroke:#e94560,color:#eee
    style F4 fill:#16213e,stroke:#e94560,color:#eee
    style F5 fill:#16213e,stroke:#e94560,color:#eee
```

<details>
<summary><b>Expand: Detailed Failure Point Analysis</b></summary>

| # | Failure Point | Observed Behavior | Operational Consequence |
|:-:|:---|:---|:---|
| 1 | **Temporal Decay** | BMD Signal 10 traverses a 6-tier bureaucratic relay (BMD → DDM → Division → District → Upazila → Union → Community). Each node introduces latency. | During Cyclone Mocha (May 2023), communities in southern Cox's Bazar received actionable information 14+ hours after BMD's initial Signal 10 issuance. |
| 2 | **Semantic Mismatch** | Warnings employ meteorological nomenclature (knots, hectopascals, signal numbers). Generic advisories ("stay alert") carry no location-specific intelligence. | A subsistence fisherman and a school headteacher receive identical messages, though their required protective actions differ fundamentally. |
| 3 | **Channel Homogeneity** | Warnings are disseminated through a uniform channel mix regardless of local telecommunication infrastructure. | Chars and islands with no mobile coverage, intermittent electricity, and seasonal road severance receive the same SMS-dependent approach as Dhaka. |
| 4 | **Impact Opacity** | Warnings describe the hazard ("a cyclone is approaching"), not the projected impact ("your school will flood; your nearest shelter is 24,000 persons over capacity"). | Local decision-makers lack quantified evidence to justify early, costly anticipatory actions such as school closure or shelter pre-positioning. |
| 5 | **No Child Protocol** | No standardized, automated protocol exists for school-level anticipatory action. School-as-shelter dual use creates an unresolved operational paradox. | School closure decisions are ad hoc. Children are either dismissed too late or schools close unnecessarily, eroding institutional trust. |

</details>

**Sources:** IFRC Cyclone Mocha DREF Final Report (MDRBD030), WMO Impact-Based Forecast and Warning Services framework, ReliefWeb situation updates, BTRC/ITU connectivity data.

---

## System Overview

InstaWarn is **middleware**: a seven-module automated pipeline that sits between the national meteorological forecast and the community-level protective action. It ingests a BMD/FFWC advisory and produces hyperlocal, audience-specific, channel-routed warning messages in Bangla, with a dedicated child-safety protocol.

### Alignment with Mapathon 2026 Challenge Tracks

```mermaid
mindmap
  root((InstaWarn))
    Track 1: ResilienceAI
      Map critical infrastructure exposure
        M1 M2 M3: Spatial hazard footprints<br/>intersected with OSM infrastructure<br/>and WorldPop demographics
      Identify vulnerable communities
        Vulnerability Index: poverty,<br/>shelter deficit, child ratio,<br/>population density, connectivity
      Automated risk assessment pipelines
        Entire pipeline is deterministic code<br/>Open data, one-command execution<br/>Git-versioned, fully reproducible
    Track 2: Last-Mile
      Overcome channel constraints
        M4: Audience-specific Bangla warnings<br/>across SMS, IVR, community radio,<br/>WhatsApp, loudspeaker
      Coordinate local authorities
        M7: Decision Dashboard with shared<br/>operational picture for local officials
      Child-centred anticipatory action
        M6: School Safety Protocol with phased<br/>school closure and shelter transition
```

---

## System Architecture

### End-to-End Data Flow: Forecast to Protective Action

```mermaid
flowchart TD
    Forecast["<b>BMD / FFWC Forecast</b><br/><i>National meteorological advisory</i>"]

    subgraph SPATIAL["Spatial Analysis Layer"]
        direction TB
        M1["<b>M1: Hazard Engine</b><br/>Cyclone · Flood · Landslide<br/>─────────────────<br/>IBTrACS track + SRTM DEM →<br/>union-level severity zonation<br/><i>Output: impact_zone.geojson</i><br/>(EXTREME / HIGH / MODERATE / LOW)"]
        M2["<b>M2: Exposure Engine</b><br/>Infrastructure · Population<br/>─────────────────<br/>OSM Overpass API + WorldPop 100m + BBS/DDM<br/>Spatial join: assets and populations<br/>within each hazard polygon<br/><i>Output: impact_profiles.json</i>"]
        M3["<b>M3: Risk Scoring</b><br/>UNDRR Composite Formula<br/>─────────────────<br/>R = H(severity) × E(density) × V(socioeconomic)<br/>Prioritized union list with<br/>four-tier composite classification"]
    end

    subgraph ACTION["Action Layer"]
        direction TB
        M4["<b>M4: Warning Generator</b><br/><i>Gemini 1.5-Flash</i><br/>7 audiences × 5 channels<br/>= 35 Bangla warning variants"]
        M5["<b>M5: NLP Triage</b><br/><i>Bidirectional</i><br/>Inbound crisis text parsing<br/>+ fuzzy geocoding"]
        M6["<b>M6: School Safety Protocol</b><br/><i>Phased State Machine</i><br/>Closure → Transition →<br/>Shelter Activation"]
    end

    M7["<b>M7: Decision Dashboard</b><br/><i>Operational Interface</i><br/>Streamlit + Folium · 7 interactive pages<br/>Situation map, disaster replay, warning journey,<br/>impact intelligence, AI warning gen, school monitor"]

    Forecast --> M1
    M1 --> M2
    M2 --> M3
    M3 --> M4
    M3 --> M5
    M3 --> M6
    M4 --> M7
    M5 --> M7
    M6 --> M7

    style Forecast fill:#0d1b2a,stroke:#00b4d8,stroke-width:2px,color:#e0e0e0
    style SPATIAL fill:#1b263b,stroke:#00b4d8,stroke-width:1px,color:#e0e0e0
    style ACTION fill:#1b263b,stroke:#48cae4,stroke-width:1px,color:#e0e0e0
    style M7 fill:#0d1b2a,stroke:#90e0ef,stroke-width:2px,color:#e0e0e0
```

### Logical Decomposition

```mermaid
block-beta
    columns 7
    block:header:7
        columns 7
        H["InstaWarn: Seven-Module Pipeline"]
    end
    M1["M1\nHazard\nEngine"]:1
    M2["M2\nExposure\nEngine"]:1
    M3["M3\nRisk\nScoring"]:1
    M4["M4\nWarning\nGenerator"]:1
    M5["M5\nNLP\nTriage"]:1
    M6["M6\nSchool\nProtocol"]:1
    M7["M7\nDashboard"]:1

    style header fill:#0d1b2a,stroke:#00b4d8,color:#e0e0e0
    style M1 fill:#16213e,stroke:#e94560,color:#eee
    style M2 fill:#16213e,stroke:#f0a500,color:#eee
    style M3 fill:#16213e,stroke:#00b4d8,color:#eee
    style M4 fill:#16213e,stroke:#48cae4,color:#eee
    style M5 fill:#16213e,stroke:#90e0ef,color:#eee
    style M6 fill:#16213e,stroke:#a8dadc,color:#eee
    style M7 fill:#16213e,stroke:#caf0f8,color:#eee
```

---

## Module Specifications

### M1: Hazard Engine

| Attribute | Detail |
|:---|:---|
| **Purpose** | Generate union-level hazard severity polygons from meteorological forecast data |
| **Cyclone Model** | Elliptical wind-radius decay model applied against coastal DEM; storm surge inundation depth estimated via SRTM 30m elevation thresholds |
| **Flood Model** | Gauge-based river stage exceedance mapped against DEM-derived floodplain delineation |
| **Landslide Model** | Slope gradient (derived from SRTM) weighted by antecedent rainfall intensity |
| **Toolchain** | GeoPandas, rasterio, Shapely |
| **Output** | `impact_zone_{timestep}.geojson` with union polygons classified into four severity tiers |

### M2: Exposure Engine

| Attribute | Detail |
|:---|:---|
| **Purpose** | Quantify infrastructure assets and population within each hazard polygon |
| **Method** | Automated `sjoin()` of hazard polygons with OSM infrastructure layers (schools, hospitals, shelters, roads) and WorldPop 100m population raster via `zonal_stats()` |
| **Toolchain** | GeoPandas, rasterstats, Overpass API |
| **Output** | `impact_profiles.json` with per-union infrastructure counts, population totals, and shelter capacity figures |

### M3: Risk Scoring

| Attribute | Detail |
|:---|:---|
| **Purpose** | Compute composite risk scores per union using the UNDRR framework |
| **Formula** | `R = H(severity, temporal_decay) × E(infrastructure_density, shelter_gap) × V(poverty_index, child_ratio, connectivity_score)` |
| **Toolchain** | NumPy, Pandas |
| **Output** | Ranked union list with four-tier classification (EXTREME, HIGH, MODERATE, LOW) |

### M4: Warning Generator

| Attribute | Detail |
|:---|:---|
| **Purpose** | Produce localized, audience-specific warning messages in Bangla |
| **Method** | Gemini 1.5-Flash under constrained system prompts; receives structured impact data, returns natural-language Bangla messages |
| **Audience Matrix** | 7 audiences (fishermen, farmers, headteachers, parents, CPP volunteers, local officials, general public) × 5 channels (SMS, IVR, community radio, WhatsApp, loudspeaker) = **35 warning variants per union** |
| **Toolchain** | google-generativeai |

### M5: NLP Triage

| Attribute | Detail |
|:---|:---|
| **Purpose** | Parse inbound crisis reports into structured incident records with geolocation |
| **Method** | Inbound text parsed into Pydantic schema (incident type, urgency level, headcount, resource requirement); location resolved via Levenshtein distance fuzzy matching against the OSM gazetteer |
| **Toolchain** | thefuzz, Gemini (ETL parser only) |
| **Output** | Live structured incident GeoJSON overlay on the decision dashboard |

### M6: School Safety Protocol

| Attribute | Detail |
|:---|:---|
| **Purpose** | Manage the phased transition of school buildings from educational use to emergency shelter function |
| **Method** | Deterministic finite state machine with time-gated transitions (see [State Machine Diagram](#the-school-shelter-paradox)) |
| **Output** | Real-time school status feed + shelter availability tracker with overflow redirection |

### M7: Decision Dashboard

| Attribute | Detail |
|:---|:---|
| **Purpose** | Provide a shared operational picture for local decision-makers |
| **Pages** | Situation Map · Disaster Replay · Warning Journey · Impact Intelligence · AI Warning Generator · School Monitor · Multi-Hazard Proof |
| **Toolchain** | Streamlit, Folium, streamlit-folium |

---

## Why Programmatic Pipelines, Not Desktop GIS

InstaWarn performs the same spatial operations as ArcGIS or QGIS (spatial joins, zonal statistics, buffer analysis, raster-vector intersection) but executes them through `GeoPandas`, `Shapely`, `rasterio`, and `rasterstats` in code.

```mermaid
quadrantChart
    title Operational Comparison: Desktop GIS vs. Programmatic Pipeline
    x-axis "Low Reproducibility" --> "High Reproducibility"
    y-axis "Low Throughput" --> "High Throughput"
    quadrant-1 "Optimal: InstaWarn"
    quadrant-2 "Fast but fragile"
    quadrant-3 "Manual and slow"
    quadrant-4 "Reproducible but slow"
    "Desktop GIS (manual)": [0.2, 0.15]
    "Semi-automated GIS": [0.45, 0.4]
    "InstaWarn Pipeline": [0.88, 0.85]
    "Script-based (no framework)": [0.7, 0.55]
```

| Dimension | Desktop GIS Workflow | InstaWarn Pipeline |
|:---|:---|:---|
| **Reproducibility** | Analyst-dependent click sequences; not version-controlled | Deterministic code; identical input produces identical output |
| **Throughput** | Hours per event (manual layer loading, styling, export) | 76 seconds end-to-end |
| **Scalability** | One district at a time | Configuration-driven; substitute district or hazard type via parameter |
| **Output Scope** | Static maps | Maps + localized warnings + school protocols + triage feeds |
| **Auditability** | Project file on one workstation | Open-source Git repository; any party can clone, inspect, and verify |

For anticipatory action within a 48 to 72 hour warning window, automated processing is not optional: it is a prerequisite.

---

## Validation: Cyclone Mocha 2023 Hindcast

Hindcasting (retrospective application of a system to a historical event using empirical data) is the standard validation methodology for early warning systems, endorsed by both WMO and ECMWF. InstaWarn replays Cyclone Mocha (May 12-14, 2023) using real datasets to demonstrate that, given this input, these outputs are produced deterministically.

### Data Inputs

| Input | Source | Spatiotemporal Resolution | License |
|:---|:---|:---|:---|
| Cyclone track and intensity | IBTrACS (NCEI, NOAA) | 6-hourly positions | Public Domain |
| Administrative boundaries | geoBoundaries via HDX | ADM4 (Union) | ODC-ODbL |
| Infrastructure | OpenStreetMap via Overpass API | Individual features | ODbL |
| Population density | WorldPop Constrained 2020 | 100m grid | CC-BY 4.0 |
| Elevation | NASA SRTM V003 | 30m (1 arc-second) | Public Domain |

> **Note:** The temporal sequencing of system actions is simulated (InstaWarn did not exist during Mocha). The spatial analysis operates on real geospatial data; the pipeline demonstrates what outputs it would have generated given the same meteorological inputs.

### Hindcast Results at T-48h

| Metric | Value |
|:---|:---|
| Unions classified HIGH/EXTREME | 32 (Coastal Cox's Bazar) |
| Population within impact zone | 638,692 |
| Schools triggered for closure protocol | 84 |
| Shelters activated | 38 standard + 9 dual-purpose school-shelters |
| Warning variants generated | 35 (7 audiences × 5 channels) |
| Pipeline execution time | 76 seconds |

### Warning Dissemination Timeline: Observed vs. InstaWarn

```mermaid
flowchart LR
    subgraph OBS["Mocha 2023 (Observed Latency)"]
        direction LR
        O1["T-48h<br/>BMD Signal 10"]
        O2["T-43h<br/>District Office<br/><b>+5h delay</b>"]
        O3["T-40h<br/>Upazila Activation<br/><b>+8h delay</b>"]
        O4["T-34h<br/>Loudspeaker<br/><b>+14h delay</b>"]
        O5["T-6h<br/>Chars Reached<br/><b>+42h delay</b>"]
        O6["Never<br/>School Closure"]
        O1 --> O2 --> O3 --> O4 --> O5 --> O6
    end

    subgraph IW["InstaWarn (Hindcast)"]
        direction LR
        I1["T-48h<br/>BMD Signal 10<br/>Ingested"]
        I2["T-48h<br/>All Channels<br/><b>Instant</b>"]
        I3["T-48h<br/>School Protocol<br/><b>Instant</b>"]
        I4["T-48h<br/>Chars Reached<br/><b>Instant</b>"]
        I1 --> I2
        I1 --> I3
        I1 --> I4
    end

    style OBS fill:#1a1a2e,stroke:#e94560,stroke-width:2px,color:#eee
    style IW fill:#1a1a2e,stroke:#00b4d8,stroke-width:2px,color:#eee
    style O1 fill:#16213e,stroke:#e94560,color:#eee
    style O2 fill:#16213e,stroke:#e94560,color:#eee
    style O3 fill:#16213e,stroke:#e94560,color:#eee
    style O4 fill:#16213e,stroke:#e94560,color:#eee
    style O5 fill:#16213e,stroke:#e94560,color:#eee
    style O6 fill:#4a0000,stroke:#e94560,color:#eee,stroke-width:2px
    style I1 fill:#16213e,stroke:#00b4d8,color:#eee
    style I2 fill:#0a3d2a,stroke:#00b4d8,color:#eee
    style I3 fill:#0a3d2a,stroke:#00b4d8,color:#eee
    style I4 fill:#0a3d2a,stroke:#00b4d8,color:#eee
```

| Milestone | Mocha 2023 (Observed) | InstaWarn (Hindcast) | Latency Reduction |
|:---|:---|:---|:---|
| BMD Signal 10 issued | T-48h | T-48h (same input) | Baseline |
| District office notified | T-43h (~5h delay) | T-48h (instantaneous) | 5 hours |
| Upazila activation begins | T-40h (~8h delay) | T-48h (instantaneous) | 8 hours |
| First community loudspeaker | T-34h (~14h delay) | T-48h (instantaneous) | 14 hours |
| School closure notification | Never (ad hoc) | T-48h (automated) | ∞ → 0 |
| Chars and islands reached | T-6h or never | T-48h (radio + CPP routing) | 42+ hours |

---

## The School-Shelter Paradox

Bangladesh has approximately 4,000 designated cyclone shelters. A significant proportion are school buildings designed for dual use. This creates a direct operational conflict:

- The school must **close** to dismiss children safely (requires maximum lead time)
- The school must **open** as a shelter to receive evacuees (requires rapid activation)

No existing system manages this transition programmatically. Module 6 implements a **deterministic finite state machine** with time-gated phase transitions:

```mermaid
stateDiagram-v2
    [*] --> NORMAL

    NORMAL --> ALERT : BMD advisory received<br/>School in HIGH/EXTREME zone
    ALERT --> PARENT_NOTIFY : T-36h<br/>Bulk SMS to parents<br/>with pickup instructions
    PARENT_NOTIFY --> CLOSURE_CHECK : T-24h<br/>Closure confirmation<br/>window opens

    state CLOSURE_CHECK {
        [*] --> PENDING
        PENDING --> CONFIRMED : Headteacher confirms
        PENDING --> ESCALATED : Overdue → UNO notified
        ESCALATED --> CONFIRMED : UNO confirms
    }

    CLOSURE_CHECK --> SHELTER_TRANSITION : Closure confirmed<br/>12-hour buffer begins

    SHELTER_TRANSITION --> SHELTER_ACTIVE : T-12h<br/>Shelter accepting evacuees

    state SHELTER_ACTIVE {
        [*] --> ACCEPTING
        ACCEPTING --> CAPACITY_REACHED : Occupancy = Capacity
        CAPACITY_REACHED --> OVERFLOW_REDIRECT : Redirect to nearest<br/>alternative shelter
    }

    SHELTER_ACTIVE --> STAND_DOWN : All-clear issued
    STAND_DOWN --> NORMAL : Facility restored<br/>to educational use

    note right of ALERT
        SMS sent to headteacher
        with school-specific
        hazard exposure data
    end note

    note right of SHELTER_ACTIVE
        Real-time occupancy
        tracking with overflow
        redirection logic
    end note
```

This protocol directly serves the competition's funding context: the GFFO Child-Centred Anticipatory Action project, implemented by Save the Children.

---

## Channel Routing for Isolated Communities

Expert practitioners rightly ask: how does InstaWarn reach communities that are truly isolated, not merely underserved, but physically separated, with structural telecommunication deficits?

InstaWarn's answer is the **Channel Routing Engine** (embedded in M4), which assigns the optimal dissemination channel per union based on that union's connectivity profile:

```mermaid
flowchart LR
    INPUT["Union Connectivity<br/>Profile Assessment"]

    INPUT --> C1{"4G coverage +<br/>smartphone<br/>penetration > 50%?"}
    INPUT --> C2{"2G coverage +<br/>feature phones<br/>dominant?"}
    INPUT --> C3{"No reliable<br/>mobile<br/>coverage?"}
    INPUT --> C4{"Char / island +<br/>seasonal road<br/>severance?"}

    C1 -->|Yes| CH1["📱 SMS + WhatsApp<br/>(with map attachment)"]
    C2 -->|Yes| CH2["📞 IVR Voice Call<br/>(pre-recorded Bangla, 30s)"]
    C3 -->|Yes| CH3["📻 Community Radio<br/>+ CPP Volunteer<br/>Loudspeaker Route"]
    C4 -->|Yes| CH4["📻 Community Radio<br/>(battery-powered receivers)<br/>+ CPP Walking Route"]

    style INPUT fill:#0d1b2a,stroke:#00b4d8,color:#e0e0e0
    style CH1 fill:#16213e,stroke:#48cae4,color:#eee
    style CH2 fill:#16213e,stroke:#48cae4,color:#eee
    style CH3 fill:#16213e,stroke:#48cae4,color:#eee
    style CH4 fill:#16213e,stroke:#48cae4,color:#eee
```

The routing logic is fully functional in the prototype. The precision of the connectivity input data remains the constraint, and that limitation is documented transparently in the section below.

For truly isolated communities, the critical infrastructure is not the mobile handset. It is the **CPP volunteer network** (60,000+ volunteers operating across coastal Bangladesh) and **community radio stations**. InstaWarn generates content formatted for these channels: 20-second loudspeaker scripts and 60-second radio announcements, in natural Bangla, with location-specific shelter directions.

---

## Data Limitations and Operational Honesty

InstaWarn's prototype uses the best available open data. It does not claim this data is complete. This section documents known gaps, because exposing where data quality degrades is itself operationally valuable for any system intended for field deployment.

<details>
<summary><b>Expand: Known Data Gaps and Mitigation Strategies</b></summary>

| Data Layer | Known Gap | Current Mitigation | Requirement for Full Deployment |
|:---|:---|:---|:---|
| **OSM Infrastructure** | Incomplete at union level. Road segments may be absent or geometrically misaligned. Shelter records are sparse in rural unions. | Pipeline consumes available OSM data and flags unions with anomalously low infrastructure counts as "data-deficient." | Ground-truthed infrastructure registry maintained through existing disaster management committee structures. Accuracy validation against satellite imagery (Google Earth Engine). |
| **Shelter Capacity** | DDM shelter inventory is neither fully digitized nor publicly accessible. Many shelters lack recorded capacity figures. | Where capacity is unknown, the system applies a conservative default estimate (800 persons per shelter) and flags the assumption. Demand-supply analysis operates on available data with explicit uncertainty markers. | Digitized, web-accessible DDM shelter database with field-verified capacities. Community-level data enrichment in high-risk zones. |
| **Connectivity Profiles** | Union-level mobile coverage, smartphone penetration, and community radio presence are approximated from BTRC aggregate statistics and OpenCelliD tower geolocation, not measured per union. | Channel routing uses best available proxy indicators. The routing logic is architecturally sound; the input precision is limited. | BTRC/operator-level union coverage maps. Field survey of communication channel availability per union. |
| **Population Dynamics** | WorldPop provides static residential population estimates. It does not account for seasonal labor migration (e.g., fishermen relocating to the coast), displaced populations, or institutional populations. | Static population count used. No temporal adjustment. | Integration with UNHCR/IOM displacement datasets for areas with refugee populations. Seasonal migration models for coastal livelihoods. |

</details>

> **This is not a weakness to conceal. Any system that claims to operate at union-level granularity must confront these data quality gaps.** InstaWarn's architecture is designed so that when higher-fidelity data becomes available (from LGED, DDM, field surveys, or community mapping initiatives) it integrates directly into the existing pipeline without architectural modification. The modules consume structured inputs; improve the input fidelity, and the output quality improves proportionally.

---

## Technical Stack

```mermaid
flowchart TB
    subgraph GEO["Geospatial Engine"]
        G1[GeoPandas]
        G2[Shapely]
        G3[Fiona]
        G4[PyProj]
    end

    subgraph RASTER["Raster Analysis"]
        R1[rasterio]
        R2[rasterstats]
    end

    subgraph VIZ["Visualization"]
        V1[Folium]
        V2[streamlit-folium]
        V3[Matplotlib]
    end

    subgraph APP["Application"]
        A1[Streamlit]
    end

    subgraph AI["AI / NLP"]
        AI1["Gemini 1.5-Flash<br/>(google-generativeai)"]
        AI2["thefuzz<br/>(Levenshtein distance)"]
    end

    GEO --> RASTER
    GEO --> VIZ
    RASTER --> VIZ
    AI --> APP
    VIZ --> APP

    style GEO fill:#16213e,stroke:#00b4d8,color:#eee
    style RASTER fill:#16213e,stroke:#48cae4,color:#eee
    style VIZ fill:#16213e,stroke:#90e0ef,color:#eee
    style APP fill:#0d1b2a,stroke:#caf0f8,stroke-width:2px,color:#eee
    style AI fill:#16213e,stroke:#e94560,color:#eee
```

Built entirely on free, open-source tooling. Deployable at a total cost under $200/year (Gemini API costs for warning generation). No proprietary GIS licenses required.

---

## Project Structure

```
InstaWarn/
├── run_pipeline.py              ← One-command orchestrator (all 7 modules)
├── app.py                       ← Streamlit dashboard entry point
├── requirements.txt
├── .streamlit/
│   ├── config.toml
│   └── secrets.toml             ← Gemini API key (not committed)
├── src/
│   ├── config.py                ← Central path and constant registry
│   ├── data_processing/         ← Raw → processed data pipeline
│   ├── hazard_analysis/         ← Cyclone, flood, landslide zone generators
│   ├── exposure_mapping/        ← OSM overlay, vulnerability index
│   ├── impact_scoring/          ← Composite risk calculator (H × E × V)
│   ├── warning_generator/       ← Gemini-powered Bangla message generation
│   ├── inbound_triage/          ← NLP parsing, fuzzy geocoding
│   ├── school_protocol/         ← School closure / shelter transition FSM
│   └── dashboard/pages/         ← 7 Streamlit dashboard pages
├── data/
│   ├── raw/                     ← Source files (not committed; see Data Sources)
│   ├── processed/               ← Pipeline-generated intermediaries
│   └── mocha_hindcast/          ← Cyclone Mocha scenario outputs
└── outputs/
    ├── maps/                    ← Static PNGs of impact zones per timestep
    ├── sample_warnings/         ← Bangla warning text files (7 audiences)
    └── risk_scores/             ← Ranked union-level composite risk CSV
```

---

## Quick Start

```bash
# Clone and set up environment
git clone https://github.com/jubayer-ahmad/InstaWarn.git && cd InstaWarn
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Configure API access
echo 'GEMINI_API_KEY = "your_key"' > .streamlit/secrets.toml

# Execute full pipeline (76 seconds)
python run_pipeline.py --event mocha_2023

# Launch decision dashboard
streamlit run app.py
```

---

## Data Sources

| Dataset | Source | Spatiotemporal Resolution | License |
|:---|:---|:---|:---|
| Administrative Boundaries | HDX geoBoundaries | ADM4 (Union) | ODC-ODbL |
| Infrastructure | OpenStreetMap via Overpass API | Individual features | ODbL |
| Population Density | WorldPop Constrained 2020 | 100m grid | CC-BY 4.0 |
| Elevation (DEM) | NASA SRTM V003 | 30m (1 arc-second) | Public Domain |
| Cyclone Track | IBTrACS (NCEI, NOAA) | 6-hourly positions | Public Domain |
| Socioeconomic Indicators | BBS / World Bank | District / Upazila | Open Data |
| Shelter Capacities | DDM / HDX | Individual facilities | Government |

---

## Scope and Constraints

InstaWarn is explicitly bounded in scope:

- It **does not replace** BMD, FFWC, DDM, or the Cyclone Preparedness Programme.
- It **is not a finished product**. It is a functional prototype validated on one historical event (Cyclone Mocha 2023).
- It **does not claim data completeness**. It documents where data is absent and specifies what higher-fidelity data would enable.
- It **does not generate forecasts**. It consumes forecasts and translates them into actionable, localized protective intelligence.

What it demonstrates: that the translation layer, from national forecast to community protective action, can be automated, reproduced, and operated on open data at a cost any district office can sustain.

---

## Team InstaWarn

<div align="center">

| Member | Institution |
|:---|:---|
| [**Jubayer Ahmad**](https://www.linkedin.com/in/ahmadjubayer/) | IBA, University of Rajshahi |
| [**Abir Dey**](https://www.linkedin.com/in/abir-dey-798073210/) | IBA, University of Rajshahi |
| [**Md. Ashik Miah**](https://www.linkedin.com/in/ibaiteashik/) | IBA, University of Rajshahi |

📧 **Contact:** jubayerahmad.c@gmail.com · +8801797799424

</div>

---

<div align="center">
  <i>Developed for Mapathon 2026, organized by RIMES and Save the Children under the GFFO-funded Child-Centred Anticipatory Action project.</i>
  <br><br>
  <img src="https://img.shields.io/badge/Built_with-Open_Data-3DA639?style=flat-square" alt="Open Data">
  <img src="https://img.shields.io/badge/Built_with-Open_Source-181717?style=flat-square" alt="Open Source">
  <img src="https://img.shields.io/badge/Validated_on-Cyclone_Mocha_2023-e94560?style=flat-square" alt="Hindcast Validated">
</div>
