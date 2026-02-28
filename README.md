# Autonode — AI IoT Dead Zone Solver

> Industrial IoT mesh-network monitoring with Jac/Jaseci graph agents.

![Autonode Dashboard](https://img.shields.io/badge/status-live-22c55e?style=flat-square) ![Jac](https://img.shields.io/badge/backend-Jac%200.11.2-38bdf8?style=flat-square) ![React](https://img.shields.io/badge/frontend-React-61dafb?style=flat-square)

## What It Does

Autonode models an industrial warehouse as a persistent graph and runs a 5-step agent pipeline to handle sensor incidents:

1. `PulseAgent` (`HeartbeatMonitor`) detects stale/missed heartbeats.
2. `DeadzoneAgent` (`DeadZoneMapper`) maps affected sensor spread.
3. `CauseAgent` (`RootCauseAnalyzer`) identifies likely root cause.
4. `Rerout Agent` (`ReroutingAgent`) attempts mesh reroute when auto-fixable.
5. `DispatchAgent` creates work-order payloads for unresolved incidents.

## Architecture

```
React Frontend (localhost:3000)
            |
            | HTTP (REST)
            v
Jac Backend (localhost:8000)
  - server.jac (public API walkers)
  - main.jac   (graph schema + agent walkers)
```

Graph shape:

- 1 warehouse
- 4 zones
- 4 routers
- 24 sensors

## Important Behavior Notes

- Backend agents are real Jac walkers and mutate graph state when pipeline walkers are called.
- There is no always-on backend daemon loop by default; execution is request-driven (for example `api_ingest`, `api_heartbeat`, `api_run_pipeline`).
- The current React dashboard includes rich UI-side simulation timelines for incident playback; backend is still used for graph setup/state/reset and API-based flows.

## Tech Stack

- Backend: Jac/Jaseci + Python 3.11
- Frontend: React 19 (CRA)
- API: Walker endpoints exposed via `jac start`/`jac serve`
- Notifications: Slack webhook + SMTP email (optional, env-configured)

## Local Run (Recommended, No Docker Required)

### Prerequisites

- Python 3.11+
- Node.js 18+

### 1) Start Backend

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
python -m pip install -U pip
python -m pip install -r requirements.txt
jac start server.jac
```

If `jac start` is unavailable in your CLI build:

```bash
jac serve server.jac -ho 0.0.0.0 -p 8000
```

Backend URL: `http://localhost:8000`

### 2) Start Frontend

```bash
cd frontend
npm install
npm start
```

Frontend URL: `http://localhost:3000`

## API Endpoints

Most endpoints are `POST` with JSON body.

| Endpoint | Description |
|---|---|
| `/walker/api_setup` | Initialize warehouse graph (idempotent) |
| `/walker/api_state` | Get full graph snapshot |
| `/walker/api_ingest` | Ingest telemetry update for one sensor |
| `/walker/api_heartbeat` | Run heartbeat monitor pass |
| `/walker/api_simulate_dropout` | Force dropout for a sensor |
| `/walker/api_run_pipeline` | Run end-to-end incident pipeline |
| `/walker/api_diagnose` | Diagnose one sensor |
| `/walker/api_dispatch` | Create + deliver one dispatch |
| `/walker/api_business_metrics` | Runtime metrics snapshot |
| `/walker/api_notification_health` | Notification integration health |
| `/walker/api_reset` | Reset sensor state and optionally metrics |

Useful discovery endpoints:

| Endpoint | Description |
|---|---|
| `GET /walkers` | List available walkers |
| `GET /functions` | List available functions |

## Demo Flow

1. Open `http://localhost:3000`.
2. Confirm the UI shows backend connection as live.
3. Trigger a failure scenario from the control panel.
4. Inspect logs, sensor status transitions, and work-order/self-heal outcomes.
5. Click reset to return graph/UI state to baseline.

## Project Structure

```text
Autonode/
├── backend/
│   ├── main.jac              # Graph schema + core walkers
│   ├── server.jac            # Public API walkers
│   ├── automation_runtime.py # Metrics/event/work-order runtime state
│   ├── notify.py             # Slack/SMTP notifications
│   ├── fullstack.jac         # Fullstack entrypoint
│   └── jac.toml
├── frontend/
│   ├── src/App.js            # Main React dashboard
│   └── package.json
└── README.md
```

## Why Jac

- The warehouse is naturally represented as a graph.
- Agent logic maps cleanly to walkers traversing nodes/edges.
- Graph state persists across requests.
- API endpoints come directly from published walkers.

Built for the Jaseci Hackathon 2026.
