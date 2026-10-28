# 🚦 Smart City Traffic Intelligence & Adaptive Traffic Management System
### Research-Grade Microscopic Simulation Powered by Eclipse SUMO, Computer Vision, AQI & 3D Web Visualization

An integrated, research-oriented Smart City Traffic Intelligence System running primarily on **Apple Silicon (MacBook Air M4 / macOS)** combining:
1. **Realistic Microscopic Traffic Simulation**: Eclipse SUMO TraCI engine with synthetic spatial fallback.
2. **Autonomous Adaptive Signal Control**: Density and waiting-time controller with safe clearance and starvation prevention vs. fixed-time baseline.
3. **Interactive 3D Visualization**: Procedural urban intersection, multi-class vehicles, cantilever signals, and dynamic camera presets in Three.js / React Three Fiber.
4. **Geographic Road-Network Ingestion**: OpenStreetMap Leaflet interface, curated Indian urban corridors (Delhi ITO, Mumbai BKC, Bengaluru Silk Board), and OSM XML topology validator.
5. **Air Quality (AQI) & Emissions Modeling**: CPCB National Air Quality Index (NAQI) stations, 24-hour diurnal trends, and microscopic tailpipe emissions (COPERT 5 / HBEFA 4.1).
6. **CCTV Video Ingestion & Vehicle Analytics**: Privacy-preserving OpenCV vehicle detection, centroid tracking, directional counting lines, and demand calibration.
7. **Rule-Based Traffic Incident Detection**: Detects sudden queue growth, stopped vehicle obstructions on green, and lane blockages with operator workflow.
8. **Emergency Vehicle Priority Preemption**: Configurable ambulance scenario with safe transition clearance.
9. **Experiment Laboratory**: SQLite persistent experiment runner, multi-scenario comparisons, and publication-ready CSV exports.

---

## 🌟 Key Architecture & Capabilities

### 1. Eclipse SUMO Microscopic Simulation Engine
- **Authoritative Ground Truth**: Eclipse SUMO simulates vehicle movement, Krauss car-following kinematics, lane assignments, junction conflict resolution, and signal state execution.
- **Heterogeneous Indian Urban Traffic Mix**: Calibrated vehicle classes matching typical Indian urban conditions:
  - **Two-Wheelers** (Motorcycles, scooters — nimble footprint, rapid acceleration)
  - **Auto-Rickshaws** (Three-wheelers — moderate speed, compact geometry)
  - **Passenger Cars** (Sedans/SUVs)
  - **City Transit Buses** (11.0m length)
  - **Commercial Heavy Trucks** (9.0m length)
  - **Bicycles** (1.8m length)
  - **Emergency Ambulances** (Preemptive high-priority vehicles)
- **Continuous 3D Coordinate Mapping**: Vehicle positions, speeds, lane IDs, and headings are extracted via TraCI every step and projected into Three.js 3D world space:
  $$\text{world}_x = \text{sumo}_x - 150.0$$
  $$\text{world}_z = -(\text{sumo}_y - 150.0)$$
  $$\text{heading\_deg} = (180.0 - \text{sumo\_angle}) \pmod{360.0}$$
- **Zero Hallucinated Physics**: The frontend only renders vehicles and signals received from SUMO through TraCI; it does not invent vehicle trajectories or traffic light phases independently.

### 2. Autonomous Adaptive Signal Controller
- **Traffic Engineering Bounds**:
  - Enforces minimum green duration ($15.0\text{s}$) to avoid driver confusion and pedestrian entrapment.
  - Enforces maximum green duration ($65.0\text{s}$) to prevent cross-street starvation.
  - Safe clearance sequence: Yellow clearance interval ($3.0\text{s}$) followed by an All-Red inter-green clearance ($1.0\text{s}$) before granting opposing green.
  - Conflicting green movements are strictly prohibited by both network geometry and state-machine mutual exclusion.
- **Hysteresis Anti-Oscillation**: Requires cross-traffic pressure to exceed active green pressure by a configured hysteresis threshold ($\times 1.35$) and queue difference before initiating a phase switch, preventing rapid toggling.
- **Emergency Preemption**: Real-time preemption clears opposing greens upon detection of priority emergency vehicles.
- **Explainable Audit Log**: Every decision (HOLD, TRANSITION) records timestamp, queue counts, accumulated wait times, and the engineering rationale.

### 3. Geographic Road Ingestion & OpenStreetMap Map Layer
- Leaflet map view styled with CartoDB Dark Matter tiles and OpenStreetMap attribution.
- Curated Indian urban arterials: **Delhi ITO Junction**, **Mumbai BKC Connector**, and **Bengaluru Silk Board**.
- Map marker layers for signalized junctions, CCTV cameras, and air-quality monitoring stations.
- In-browser OSM XML network validator assessing `<node>` topology, `<way>` segments, and highway attributes for SUMO `netconvert` compilation.

### 4. Air Quality (AQI) & Fleet Emissions Modeling
- Provider-adapter architecture supporting CPCB reference stations and OpenAQ API v3.
- Official Indian Central Pollution Control Board (CPCB) NAQI formula calculating sub-indices for PM2.5, PM10, NO2, SO2, CO, and O3 with standard color-coding.
- 24-hour diurnal pollutant trend charts.
- Microscopic vehicle tailpipe emissions engine calculating instantaneous rates (CO₂ kg/h, NOx g/h, PM g/h, fuel l/h, cumulative CO₂) using COPERT 5 / HBEFA 4.1 factors with speed-dependent idling penalties.
- Explicit scientific disclosure distinguishing modeled tailpipe estimates from ambient physical air sensor measurements.

### 5. CCTV Video Ingestion & Computer Vision Analytics
- Multipart video upload supporting MP4, MOV, and AVI recordings with size limits.
- Background asynchronous OpenCV processing using MOG2 background subtraction and Euclidean Centroid Tracking.
- Directional virtual counting lines and vehicle classification (two-wheeler, auto-rickshaw, car, bus/truck).
- Calibrated speed and stopline queue-length estimation.
- Camera-to-SUMO approach mapping interface (North, South, East, West).
- Demand calibration toggle allowing camera-observed traffic densities to adjust simulated controller demand.
- Privacy-preserving architecture: Strictly NO facial recognition or license-plate storage.

### 6. Traffic Incident Detection & Emergency Preemption
- Explainable rule-based anomaly detectors:
  - Sudden queue surges ($\ge 5$ vehicle accumulation spike)
  - Stationary vehicle obstruction during active green phases ($>25\text{s}$)
  - Abnormal speed collapses and travel lane blockages
- Operator workflow: Detected $\to$ Confirmed $\to$ Resolved or Dismissed (false positive).
- Operator anomaly injection tool to test adaptive controller rerouting and delay accumulation.
- Dedicated emergency vehicle preemption scenario testing travel time improvements.

### 7. Experiment Laboratory & SQLite Persistence
- Repeatable multi-scenario benchmark runner comparing Fixed-Time vs. Adaptive Density control under identical seeds and demand levels.
- SQLite database (`data/smart_traffic.db`) storing all experiment runs, configurations, and metrics.
- Publication-ready CSV report generation and download.


---

## 🏗️ System Architecture

```
traffic-intersection-intelligence/
├── backend/                               # FastAPI + WebSockets Backend
│   ├── main.py                            # ASGI application & lifespan lifecycle
│   ├── api/
│   │   ├── routes.py                      # REST endpoints & CSV export
│   │   └── websocket.py                   # /ws/simulation streaming
│   ├── models/
│   │   └── schemas.py                     # Pydantic schemas (Vehicle3D, SignalState3D, etc.)
│   ├── services/
│   │   ├── simulation_service.py          # Unified SUMO / Synthetic orchestrator
│   │   └── spatial_engine.py              # Spatial fallback engine
│   └── tests/                             # API, WebSocket, and robustness tests (21 tests)
│
├── traffic-intelligence-system/           # Core Python package & SUMO files
│   ├── src/traffic_intel/
│   │   ├── sumo/                          # SUMO microscopic network definitions
│   │   │   ├── intersection.nod.xml       # Intersection nodes & priority points
│   │   │   ├── intersection.edg.xml       # 2-lane arterial road links
│   │   │   ├── intersection.net.xml       # Compiled SUMO network (netconvert)
│   │   │   ├── routes.rou.xml             # Heterogeneous vehicle types & calibrated flows
│   │   │   ├── simulation.sumocfg         # Authoritative SUMO configuration
│   │   │   └── NETWORK_SPECIFICATION.md   # Mapped facts vs synthetic assumptions
│   │   ├── sumo_adapter.py                # TraCI communication adapter (SumoSimulator)
│   │   ├── sumo_controller.py             # SumoAdaptiveController & FixedTimeController
│   │   ├── models.py                      # Vehicle and direction data models
│   │   ├── city_profiles.py               # 7 Indian urban profiles
│   │   ├── signal_control.py              # Signal phase logic
│   │   ├── simulator.py                   # Original simulation core
│   │   └── analytics.py                   # Congestion analysis
│   └── tests/                             # Unit tests (61 tests)
│
├── frontend/                              # React 19 + TypeScript + Vite + Three.js
│   ├── src/
│   │   ├── components/                    # 3D Viewport, Road, Signals, Vehicles
│   │   ├── dashboard/                     # Telemetry cards, Control deck, Modal
│   │   ├── hooks/                         # useSimulationSocket (WebSocket & REST)
│   │   └── types/                         # TypeScript interfaces
│   └── package.json
│
└── data/experiments/                      # Exported CSV experiment results
```

---

## 🚀 Quickstart & Installation (macOS Apple Silicon & Linux)

### Prerequisites
- **macOS** (Apple Silicon M1/M2/M3/M4) or Linux
- **Python 3.11+** (Python 3.13 supported)
- **Node.js 18+** and **npm**

### Step 1: Python Environment & SUMO Installation

Eclipse SUMO provides native Apple Silicon arm64 wheels on PyPI:

```bash
cd traffic-intelligence-system
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies including native Eclipse SUMO, TraCI, and sumolib
pip install eclipse-sumo sumo-data sumolib traci
pip install -e .
pip install -r requirements.txt
pip install fastapi "uvicorn[standard]" websockets pytest pytest-asyncio
```

### Step 2: Verify SUMO Installation (Phase A Proof of Concept)

Run the standalone verification demo:

```bash
python traffic-intelligence-system/src/traffic_intel/sumo_adapter.py
```

Expected output:
```
=== SUMO Integration Verification Demo ===
Starting SUMO simulation engine...
SUMO started! Advancing 50 steps (step_length=0.1s)...
Step 010 (Time 1.0s): Vehicles=14, Queues={'north': 0, 'south': 0, 'east': 0, 'west': 0}, ActivePhase=NS_GREEN, MeanSpeed=42.8 km/h
...
Closing simulation cleanly...
SUMO Proof-of-Concept verification successful!
```

### Step 3: Run the Full Test Suite (82 Tests)

Run all tests across both backend and core packages:

```bash
PYTHONPATH=. traffic-intelligence-system/.venv/bin/pytest backend/tests/ traffic-intelligence-system/tests/
```
All **82 tests** will execute and pass in ~30 seconds.

### Step 4: Start the Backend Server

```bash
# From workspace root:
PYTHONPATH=. traffic-intelligence-system/.venv/bin/uvicorn backend.main:app --host 127.0.0.1 --port 8000 --reload
```
API Documentation will be available at: `http://127.0.0.1:8000/docs`

### Step 5: Start the 3D Frontend

In a separate terminal:

```bash
cd frontend
npm install
npm run dev -- --host 127.0.0.1 --port 5173
```
Open your browser at: `http://127.0.0.1:5173`

---

## 🔬 Research Experiments & Empirical Validation

To execute a controlled experiment from the command line:

```bash
PYTHONPATH=. traffic-intelligence-system/.venv/bin/python -c "
from backend.services.simulation_service import simulation_service
res = simulation_service.run_sumo_comparison(duration_s=120.0, seed=42)
print('Fixed throughput/hr:', res.fixed.throughput_veh_per_hr, 'Mean wait:', res.fixed.mean_waiting_time_s)
print('Adaptive throughput/hr:', res.adaptive.throughput_veh_per_hr, 'Mean wait:', res.adaptive.mean_waiting_time_s)
print('Reduction in delay:', res.delta_waiting_time_pct, '%')
print('CSV exported to:', res.csv_export_path)
"
```

Results are saved to `data/experiments/sumo_experiment_<timestamp>.csv` and can be downloaded directly from the UI or via `GET /api/simulation/export-csv`.

---

## 📜 Network Specification: Mapped Facts vs. Synthetic Assumptions

| Parameter | Type | Detail |
|---|---|---|
| **Road Width & Geometry** | Mapped Fact | Standard 3.5m lane width, 1.0m median, 150m arterial links. |
| **Microscopic Kinematics**| Mapped Fact | Krauss car-following, acceleration, braking, and headway maintenance computed by SUMO engine. |
| **Signal Exclusion** | Mapped Fact | Conflict-free signal states enforced by both SUMO TLS logic and Python state-machine. |
| **Vehicle Footprints** | Mapped Fact | Two-wheelers (2.2m), Cars (4.5m), Auto-rickshaws (3.2m), Buses (11.0m). |
| **Arrival Rates** | Calibrated Assumption | Stochastic insertion rates derived from representative Indian city survey ranges (Delhi, Mumbai, Bengaluru). Never represented as raw field loop detector counts. |

---

## 🧪 License & Attribution
- Eclipse SUMO is licensed under the [Eclipse Public License v2.0 (EPL-2.0)](https://www.eclipse.org/legal/epl-2.0/).
- The Traffic Intersection Intelligence System code is open-source under the MIT License.
# System-s-MDM
