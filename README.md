# Mini Telemetry Collector & Query API

A small **Go** service that simulates a network telemetry/monitoring microservice: it polls devices for metrics, stores them in memory, and exposes a simple HTTP API for querying state and history.  

You can treat this as a one‑week, no‑AI project to learn Go from scratch while doing something network‑relevant.

---

## Goals

- Learn core Go: **types**, goroutines, channels, interfaces, testing.  
- Build a long‑running daemon that periodically polls simulated “devices” for metrics.  
- Expose a small REST API to query device inventory, current metrics, and basic history.  
- Optionally explore gRPC once the HTTP version works.

---

## Architecture Overview

Components:

- **Config loader**  
  - Reads a config file (YAML/JSON) specifying HTTP port, poll intervals, and log level.  

- **Device inventory**  
  - Reads a `devices.json`/`devices.yaml` file with entries like: `id`, `hostname`, `ip`, `role`, `poll_interval`.  

- **Poller**  
  - Runs a goroutine per device.  
  - On each interval, “polls” the device and produces metrics (either from simulated files or real pings).  
  - Sends metrics into a central channel or directly into a metrics store.  

- **Metrics store**  
  - In‑memory map keyed by `(device_id, metric_name)` containing latest value and a small history (for example, a slice of recent samples).  

- **HTTP API**  
  - `GET /devices` – list all devices and last poll time.  
  - `GET /devices/{id}/metrics` – latest metrics.  
  - `GET /devices/{id}/metrics?since=...` – basic history since a timestamp.  

Optional later:

- **Alert engine** – evaluates conditions (for example, latency above a threshold) and maintains an in‑memory list of alerts.  
- **gRPC API** – mirrors one or two HTTP endpoints using protobuf and gRPC.

---

## Data Model

Example `devices.json`:

```json
[
  {
    "id": "edge-1",
    "hostname": "edge-1.lab",
    "ip": "192.0.2.10",
    "role": "edge",
    "poll_interval_seconds": 5
  },
  {
    "id": "core-1",
    "hostname": "core-1.lab",
    "ip": "192.0.2.20",
    "role": "core",
    "poll_interval_seconds": 10
  }
]


Current Ideal Layout:
.
├── cmd/
│   └── server/
│       └── main.go       # wire everything together
├── internal/
│   ├── config/           # config loading
│   ├── devices/          # inventory loading and device model
│   ├── poller/           # polling logic, goroutines
│   ├── metrics/          # in-memory metrics store
│   └── api/              # HTTP handlers (and optional gRPC)
├── configs/
│   ├── config.yaml
│   └── devices.json
└── README.md

