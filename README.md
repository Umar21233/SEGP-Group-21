# AIB Microalgae Monitor

An intelligent IoT monitoring and automation platform for commercial microalgae cultivation in photobioreactors. Built in collaboration with Algae International Berhad (AIB) as a second-year Software Engineering Group Project at the University of Nottingham Malaysia.

**Group 21 · COMP2019 · University of Nottingham Malaysia**

---

## What This Is

Microalgae cultivation is highly sensitive — undetected anomalies in pH, temperature, or turbidity can destroy an entire culture batch. This system replaces manual monitoring with a four-layer IoT pipeline that streams live sensor data, runs ML models for anomaly detection and parameter optimisation, and surfaces everything to operators through a role-based web dashboard.

---

## System Architecture

Four-layer pipeline: **Sensing → Acquisition → Processing → User Interface**

```
Physical Sensors (ESP32)
        ↓ MQTT
Cloud Broker
        ↓ HTTP
Firebase Firestore + Raspberry Pi (ML Engine)
        ↓
React Dashboard
```

| Layer | Technology |
|-------|-----------|
| Sensors | pH, Temperature, Light Intensity, Turbidity, Ultrasonic, Conductivity |
| Acquisition | ESP32 Microcontroller, MQTT Publisher, JSON Serialization |
| Processing | Raspberry Pi, Firebase Cloud Firestore, PPO Model, Prophet Model |
| UI | React 18 + TypeScript, Tailwind CSS, shadcn/ui, TanStack Query |

---

## ML Engine

### PPO Advisory Model (Reinforcement Learning)
- Reads all 6 live sensor readings as a continuous state vector
- Recommends adjustments to pH dosing, light level, pump rate, and stirring speed
- Turbidity increase → positive reward (biomass density growing)
- Culture stagnation or drop → negative reward (penalty applied)
- Human-in-the-loop: operators review and manually apply suggestions

### Meta Prophet Anomaly Detection
- Trains on historical turbidity data to model the normal growth curve
- Generates real-time forecasts with confidence intervals
- Flags readings outside the uncertainty band instantly
- **Negative anomaly**: turbidity below forecast — indicates crash, contamination, or hardware failure
- **Positive anomaly**: turbidity above forecast — unexpected growth spike, flags for early harvest
- One Prophet instance per PBR tank to prevent cross-tank model bias

---

## Dashboard

A role-based web dashboard with five authenticated pages:

**Analysis** — Live KPI cards for all 6 sensors, 24h trend charts, anomaly alert preview, PPO recommendation panel

**Sensors** — Per-sensor deep-dive analytics with tabbed navigation, statistical summaries (min/max/mean), configurable time range selector (1h/6h/24h/7d)

**Anomaly Alerts & Logs** — Real-time Prophet alert feed with severity badges, deviation percentages, acknowledgement workflow, and filter by type/status/time range

**Simulation** — Integrates with PPO's FastAPI endpoint on Raspberry Pi; parameter sliders generate a predicted turbidity trajectory with confidence band

**User Management** *(Admin only)* — Full CRUD for system accounts, role assignment, invite modal

### Role-Based Access Control

| Feature | Worker | Manager | Admin |
|---------|--------|---------|-------|
| Live sensor KPIs | ✓ | ✓ | ✓ |
| 24h trend charts | ✓ | ✓ | ✓ |
| Anomaly alerts | ✓ | ✓ | ✓ |
| PPO suggestions | ✓ | ✓ | ✓ |
| Simulation & ML tools | — | ✓ | ✓ |
| User management | — | — | ✓ |

---

## Backend & Data Infrastructure

- **Firebase Cloud Firestore** (NoSQL) — real-time data storage and live listener feeds
- **Broker-Subscriber MQTT pipeline** — sensor data flows ESP32 → MQTT → Firebase without loss
- **Real-time data validation & preprocessing** — asynchronous JSON ingestion with schema contracts
- **FastAPI** — serves PPO model inference endpoint on Raspberry Pi
- **Multi-site architecture** — global site switcher, per-site Firestore listeners, isolated data per tank

---

## Tech Stack

| | |
|---|---|
| Frontend | React 18, TypeScript, Vite |
| Styling | Tailwind CSS, shadcn/ui |
| State & Data | TanStack Query, React Context |
| Backend | Firebase Firestore, Firebase Auth, FastAPI |
| ML | Stable Baselines3 (PPO), Meta Prophet |
| Hardware | ESP32 Microcontroller, Raspberry Pi, 6 physical sensors |
| Pipeline | MQTT, JSON, HTTP |

---

## Running the Dashboard Locally

```bash
npm install
npm run dev
```

Create a `.env.local` file:

```
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
```

---

## Validation Results

| Component | Result |
|-----------|--------|
| Broker-Subscriber Pipeline | JSON streams correctly ESP32 → MQTT → Firebase ✓ |
| Prophet Anomaly Detection | Forecast baseline generated; anomalies flagged on live stream ✓ |
| PPO Advisory Model | Recommendations generated in test env; I/O dimensions correct ✓ |
| Dashboard & RBAC | Role rendering functional; state retained across site switches ✓ |

---

## Known Limitations

- ML models trained on synthetic turbidity data — sim-to-real gap exists with real sensor noise
- Closed-loop physical testing with full hardware stack still pending
- Cloud latency under high-frequency concurrent streams not yet stress-tested

---

## Project Context

Built for Algae International Berhad (AIB) as part of COMP2019 Software Engineering Group Project, University of Nottingham Malaysia, 2025–2026.
