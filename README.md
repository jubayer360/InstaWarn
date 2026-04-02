# InstaWarn

**Automated Impact-Based Multi-Hazard Early Warning Middleware for Bangladesh**

From forecast issuance to last-mile protective action — programmatically, reproducibly, on open data.

**Live Prototype:** [huggingface.co/spaces/jubayerahmad/InstaWarn](https://huggingface.co/spaces/jubayerahmad/InstaWarn)

**Repository:** [github.com/jubayer360/InstaWarn](https://github.com/jubayer360/InstaWarn)

---

<div align="center">
  <h2>🏆 Mapathon 2026 - 2nd Runner Up!</h2>
  <img src="https://lh3.googleusercontent.com/d/1Zp5rbvmQf0vqwZ0at85XrtHBdBPgQBYF" alt="Mapathon 2026 Prize" width="80%">
  <br><br>
  <b>We outperformed strong competitors from 29 universities to secure a spot among the top 11 finalist teams, and walked away as the 2nd Runner-up. That's something to be proud of. 🏆</b>
  <p><i>Every other finalist team came from an engineering or technical background. We were the only team with a BBA background!</i></p>
</div>

---

## The Problem InstaWarn Addresses

Bangladesh's national early warning infrastructure — BMD, FFWC, the Cyclone Preparedness Programme — is among the most mature in South Asia. InstaWarn does not replace any of it.

It addresses a specific, persistent, and documented gap: **the translation layer between a national-level forecast and a community-level protective action.**

This gap has five structural failure points, each documented in post-disaster evaluations and humanitarian after-action reviews:

| Failure Point | What Happens | Consequence |
|:---|:---|:---|
| **Time Decay** | BMD Signal 10 travels through a 6-tier bureaucratic relay (BMD → DDM → Division → District → Upazila → Union → Community). Each node adds delay. | During Cyclone Mocha (May 2023), communities in southern Cox's Bazar received actionable information 14+ hours after BMD's initial Signal 10. |
| **Language Mismatch** | Warnings use meteorological terminology (knots, hectopascals, signal numbers). Generic advisories ("stay alert") carry no location-specific intelligence. | A fisherman and a headteacher receive the same message, though their protective actions differ entirely. |
| **Channel Blindness** | Warnings are disseminated through a uniform channel mix regardless of local connectivity. | Chars and islands with no mobile coverage, no electricity, and seasonal road cuts receive the same SMS-centric approach as Dhaka. |
| **Impact Opacity** | Warnings describe the hazard ("a cyclone is coming"), not the impact ("your school will flood, your nearest shelter is 24,000 people over capacity"). | Local decision-makers lack data to justify early, costly anticipatory actions like school closure or shelter pre-positioning. |
| **No Child Protocol** | No standardized, automated protocol exists for school-level anticipatory action. School-as-cyclone-shelter dual use creates an unresolved operational paradox. | School closure decisions are ad hoc. Children are either sent home too late or schools close unnecessarily, eroding trust. |

**Sources:** IFRC Cyclone Mocha DREF Final Report (MDRBD030), WMO Impact-Based Forecast and Warning Services framework, ReliefWeb situation updates, BTRC/ITU connectivity data.

---

## What InstaWarn Is

InstaWarn is middleware — a seven-module automated pipeline that sits between the national meteorological forecast and the community-level protective action. It converts a BMD/FFWC advisory into hyperlocal, audience-specific, channel-routed warning messages in Bangla, with a dedicated child-safety protocol.

It addresses both challenge tracks of the Mapathon 2026:

| Track | Competition Requirement | InstaWarn Module |
|:---|:---|:---|
| **Track 1: ResilienceAI** | Map critical infrastructure exposure to multiple hazards | M1-M3: Spatial hazard footprints intersected with OSM infrastructure and WorldPop demographics, scored using UNDRR composite risk formula |
| **Track 1: ResilienceAI** | Identify the most vulnerable communities using spatial data | Vulnerability Index: poverty, shelter deficit, child ratio, population density, connectivity — scored per union |
| **Track 1: ResilienceAI** | Propose automated, reproducible risk assessment pipelines | Entire pipeline is code. Open data. One-command execution. Git-versioned. |
| **Track 2: Last-Mile** | Overcome channel constraints, localize warnings | M4: Audience-specific Bangla warnings across SMS, IVR, community radio, WhatsApp, loudspeaker |
| **Track 2: Last-Mile** | Coordinate local authorities and community members | M7: Decision Dashboard — shared operational picture for union chairmen, upazila officers, CPP coordinators |
| **Track 2: Last-Mile** | Child-centred anticipatory action (funding context) | M6: School Safety Protocol — phased school closure → shelter transition state machine |

---

## System Architecture

### Data Flow: Forecast to Action

```
BMD / FFWC Forecast
        │
        ▼
┌─────────────────────────┐
│  M1: Hazard Engine       │  IBTrACS track + SRTM DEM → union-level severity zones
│  Cyclone · Flood · Land. │  Output: impact_zone.geojson (EXTREME/HIGH/MODERATE/LOW)
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  M2: Exposure Engine     │  OSM Overpass API + WorldPop 100m + BBS/DDM
│  Infrastructure · Pop.   │  Spatial join: what and who is inside each hazard zone
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  M3: Risk Scoring        │  R = H(severity) × E(density) × V(socioeconomic index)
│  UNDRR Composite         │  Output: prioritized union list with composite scores
└───────────┬─────────────┘
            │
     ┌──────┼──────────────┐
     ▼      ▼              ▼
┌────────┐ ┌────────┐ ┌────────┐
│ M4:    │ │ M5:    │ │ M6:    │
│ Warning│ │ NLP    │ │ School │
│ Gen.   │ │ Triage │ │ Safety │
│(Gemini)│ │(2-way) │ │Protocol│
└───┬────┘ └───┬────┘ └───┬────┘
    └──────────┼──────────┘
               ▼
┌─────────────────────────┐
│  M7: Decision Dashboard  │  Streamlit + Folium — 7 interactive pages
│  (Operational Interface) │  Situation map, disaster replay, warning journey,
│                          │  impact intelligence, AI warning gen, school monitor
└─────────────────────────┘
```

### Module Detail

| Module | Method | Key Tools | Output |
|:---|:---|:---|:---|
| **M1: Hazard Engine** | Cyclone: elliptical wind-radius + storm surge decay against coastal DEM. Flood: gauge-based threshold against DEM. Landslide: slope × rainfall intensity. | GeoPandas, rasterio, Shapely | `impact_zone_{timestep}.geojson` — union polygons classified by severity |
| **M2: Exposure Engine** | Automated `sjoin()` of hazard polygons with OSM infrastructure (schools, hospitals, shelters, roads) and WorldPop population grid via `zonal_stats()` | GeoPandas, rasterstats, Overpass API | `impact_profiles.json` — per-union infrastructure counts, population, shelter capacity |
| **M3: Risk Scoring** | `R = H(severity, time_decay) × E(infrastructure_density, shelter_gap) × V(poverty, child_ratio, connectivity)` | NumPy, Pandas | Ranked union list with four-tier classification |
| **M4: Warning Generator** | Gemini 1.5-Flash under strict system prompts. Receives structured impact data, produces audience-specific Bangla messages. | google-generativeai | 7 audiences × 5 channels = 35 warning variants per union |
| **M5: NLP Triage** | Inbound crisis texts parsed into Pydantic schema (incident type, urgency, headcount, resource). Location fuzzy-matched via Levenshtein distance against OSM dataset. | thefuzz, Gemini (ETL parser only) | Live structured incident GeoJSON overlay |
| **M6: School Protocol** | Phased state machine: T-48h alert → T-36h parent notification → closure confirmation → shelter transition (12h buffer) → capacity tracking → overflow redirect | Custom state logic | School status feed + shelter availability tracker |
| **M7: Dashboard** | Seven Streamlit pages: Situation Map, Disaster Replay, Warning Journey, Impact Intelligence, AI Warning Generator, School Monitor, Multi-Hazard Proof | Streamlit, Folium, streamlit-folium | Operational decision interface |

---

## Why Programmatic Pipelines, Not Desktop GIS

InstaWarn performs the same spatial operations as ArcGIS or QGIS — spatial joins, zonal statistics, buffer analysis, raster-vector intersection — but executes them through `GeoPandas`, `Shapely`, `rasterio`, and `rasterstats` in code.

| Dimension | Desktop GIS | InstaWarn Pipeline |
|:---|:---|:---|
| **Reproducibility** | Analyst-dependent click sequences, not versioned | Deterministic code; identical input → identical output |
| **Speed** | Hours per event (manual layer loading, styling, export) | 76 seconds end-to-end |
| **Scalability** | One district at a time | Swap config for any district, any hazard type |
| **Output** | A map | Maps + warnings + school protocols + triage feeds |
| **Auditability** | Project file on one machine | Open-source Git repo; anyone can clone and verify |

For anticipatory action with a 48-72 hour warning window, automation is not optional.

---

## Validation: Cyclone Mocha 2023 Hindcast

Hindcasting — retrospective application of a system to a historical event using real data — is the standard validation method for early warning systems (WMO, ECMWF). InstaWarn replays Cyclone Mocha (May 12-14, 2023) using real datasets to demonstrate: given this input, these outputs are produced deterministically.

### Data Inputs

| Input | Source | Resolution | License |
|:---|:---|:---|:---|
| Cyclone track & intensity | IBTrACS (NCEI, NOAA) | 6-hourly positions | Public Domain |
| Administrative boundaries | geoBoundaries via HDX | ADM4 (Union) | ODC-ODbL |
| Infrastructure | OpenStreetMap via Overpass API | Individual features | ODbL |
| Population density | WorldPop Constrained 2020 | 100m grid | CC-BY 4.0 |
| Elevation | NASA SRTM V003 | 30m (1 arc-second) | Public Domain |

**What is simulated:** The timing of system actions (InstaWarn did not exist during Mocha). The spatial analysis uses real geospatial data; the pipeline proves what outputs it would have generated.

### Hindcast Results at T-48h

| Metric | Value |
|:---|:---|
| Unions classified HIGH/EXTREME | 32 (Coastal Cox's Bazar) |
| Population in impact zone | 638,692 |
| Schools triggered for closure protocol | 84 |
| Shelters activated | 38 standard + 9 dual-purpose school-shelters |
| Warning variants generated | 35 (7 audiences × 5 channels) |
| Pipeline execution time | 76 seconds |

### Warning Dissemination: Status Quo vs. InstaWarn

| Milestone | Mocha 2023 (Observed) | InstaWarn (Hindcast) |
|:---|:---|:---|
| BMD Signal 10 issued | T-48h | T-48h (same input) |
| District office notified | T-43h (~5h delay) | T-48h (instant) |
| Upazila activation begins | T-40h (~8h delay) | T-48h (instant) |
| First community loudspeaker | T-34h (~14h delay) | T-48h (instant) |
| School closure notification | Never (ad hoc) | T-48h (automated) |
| Chars and islands reached | T-6h or never | T-48h (radio + CPP routing) |

---

## Data Limitations and Operational Honesty

InstaWarn's prototype uses the best available open data. It does not claim this data is complete. This section documents known gaps — because exposing where data breaks is itself operationally valuable.

### Known Data Gaps

| Data Layer | Issue | How InstaWarn Handles It | What Full Deployment Requires |
|:---|:---|:---|:---|
| **OSM Infrastructure** | Incomplete at union level. Roads may be missing or misaligned. Shelter records are sparse — some unions show 1-2 shelters where dozens exist. | Pipeline uses what OSM provides and flags unions with abnormally low infrastructure counts as "data-deficient." | Ground-truthed infrastructure registry, maintained through existing disaster committee structures (ministry → union → community). Accuracy validation against satellite imagery (GEE). |
| **Shelter Capacity** | DDM shelter inventory is not fully digitized or publicly accessible. Many shelters lack recorded capacity figures. | Where capacity is unknown, the system applies a default estimate (800 per shelter) and flags it. Demand-supply analysis runs on available data with explicit uncertainty markers. | Digitized, web-accessible DDM shelter database with verified capacities. Community-level data enrichment in high-risk zones. |
| **Connectivity Profiles** | Union-level mobile coverage, smartphone penetration, and community radio presence are approximated from BTRC aggregate data and OpenCelliD tower locations, not measured per-union. | Channel routing uses best available proxies. The routing logic is sound; the input precision is limited. | BTRC/operator-level union coverage maps. Field survey of communication channel availability per union. |
| **Population Dynamics** | WorldPop provides residential population. It does not capture seasonal labor migration (fishermen moving to coast), displaced populations, or institutional populations. | Static population count used. No dynamic adjustment. | Integration with UNHCR/IOM displacement data for areas with refugee populations. Seasonal migration models for coastal livelihoods. |

**This is not a weakness to hide. Any system that claims to work at union level must confront these gaps.** InstaWarn's architecture is designed so that when better data becomes available — from LGED, DDM, field surveys, or community mapping initiatives — it plugs directly into the existing pipeline without architectural changes. The modules consume structured inputs; improve the input, and the output improves automatically.

---

## The School-Shelter Paradox

Bangladesh has approximately 4,000 designated cyclone shelters. A significant portion are school buildings designed for dual use. This creates a direct operational conflict:

- The school must **close** to send children home safely (requires maximum lead time)
- The school must **open** as a shelter to receive evacuees (requires quick activation)

No existing system manages this transition. InstaWarn's Module 6 implements a phased state machine:

```
T-48h  ─── Headteacher receives SMS alert (school in HIGH/EXTREME zone)
T-36h  ─── Parent bulk SMS: school closure + child pickup instructions
T-24h  ─── Closure confirmation check
             ├── Confirmed → proceed to shelter transition
             └── Overdue → escalate to Upazila Nirbahi Officer
T-24h  ─── Dual-use school: shelter transition timer starts
T-12h  ─── Shelter active, accepting evacuees
             Capacity tracked; overflow redirected to nearest alternative
```

This protocol directly serves the competition's funding context: the GFFO Child-Centred Anticipatory Action project, implemented by Save the Children.

---

## Reaching Isolated Communities

Expert practitioners rightly ask: how does InstaWarn reach communities that are truly isolated — not just underserved, but physically separated, with a mobile phone as the only possible medium?

InstaWarn's answer is the **Channel Routing Engine** (embedded in M4), which assigns the optimal dissemination channel per union based on connectivity profile:

| Connectivity Profile | Channel Assignment |
|:---|:---|
| 4G + smartphone penetration >50% | SMS + WhatsApp (with map attachment) |
| 2G coverage, feature phones dominant | IVR voice call (pre-recorded Bangla, 30s) |
| No reliable mobile coverage | Community radio script + CPP volunteer loudspeaker route |
| Char/island, seasonal road cut | Community radio (pre-positioned battery radio program) + CPP walking route |

The routing logic is functional in the prototype. The precision of the connectivity input data is the constraint — and that is documented honestly above.

For truly isolated communities, the critical infrastructure is not the phone. It is the **CPP volunteer network** (60,000+ volunteers operating in coastal Bangladesh) and **community radio stations**. InstaWarn generates content formatted for these channels: 20-second loudspeaker scripts and 60-second radio announcements, in natural Bangla, with location-specific shelter directions.

---

## Technical Stack

| Component | Tool | Role |
|:---|:---|:---|
| Geospatial Engine | GeoPandas, Shapely, Fiona, PyProj | Spatial joins, overlay, projection |
| Raster Analysis | rasterio, rasterstats | DEM ingestion, zonal statistics |
| Visualization | Folium, streamlit-folium, Matplotlib | Interactive web maps + static exports |
| Dashboard | Streamlit | Multipage web application |
| AI / NLP | google-generativeai (Gemini 1.5-Flash) | Warning localization + inbound triage parsing |
| Fuzzy Matching | thefuzz | Levenshtein distance for location reconciliation |

Built entirely on free, open-source tools. Deployable for under $200/year (Gemini API costs for warning generation). No proprietary GIS licenses required.

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
│   ├── school_protocol/         ← School closure / shelter transition
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
git clone https://github.com/jubayer-ahmad/InstaWarn.git && cd InstaWarn
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

echo 'GEMINI_API_KEY = "your_key"' > .streamlit/secrets.toml

python run_pipeline.py --event mocha_2023    # Full pipeline: 76 seconds
streamlit run app.py                          # Launch dashboard
```

---

## Data Sources

| Dataset | Source | Resolution | License |
|:---|:---|:---|:---|
| Administrative Boundaries | HDX geoBoundaries | ADM4 (Union) | ODC-ODbL |
| Infrastructure | OpenStreetMap via Overpass API | Individual features | ODbL |
| Population Density | WorldPop Constrained 2020 | 100m grid | CC-BY 4.0 |
| Elevation (DEM) | NASA SRTM V003 | 30m | Public Domain |
| Cyclone Track | IBTrACS (NCEI, NOAA) | 6-hourly | Public Domain |
| Socioeconomic Indicators | BBS / World Bank | District / Upazila | Open Data |
| Shelter Capacities | DDM / HDX | Individual facilities | — |

---

## What InstaWarn Is Not

- It is not a replacement for BMD, FFWC, DDM, or the Cyclone Preparedness Programme.
- It is not a finished product. It is a functional prototype validated on one historical event.
- It does not claim complete data. It documents where data is missing and what better data would enable.
- It does not generate forecasts. It consumes forecasts and translates them into actionable, localized intelligence.

What it demonstrates: that the translation layer — from national forecast to community action — can be automated, reproduced, and operated on open data at a cost any district office can sustain.

---

### Team InstaWarn

| Member | Institution |
|:---|:---|
| [Jubayer Ahmad](https://www.linkedin.com/in/ahmadjubayer/) | IBA, University of Rajshahi |
| [Abir Dey](https://www.linkedin.com/in/abir-dey-798073210/) | IBA, University of Rajshahi |
| [Md. Ashik Miah](https://www.linkedin.com/in/ibaiteashik/) | IBA, University of Rajshahi |

**Contact:** jubayerahmad.c@gmail.com | +8801797799424

---

*Developed for Mapathon 2026, organized by RIMES and Save the Children under the GFFO-funded Child-Centred Anticipatory Action project.*
