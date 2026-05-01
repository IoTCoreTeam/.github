# IoT Core Team

> **A unified core platform for identity & authentication, gateway-node orchestration, spatial management, and real-time telemetry / ACK processing over HTTP, SSE, WebSocket, and MQTT.**

---

## Project Purpose

The **IoT Core Platform** is built to solve the challenge of centralized management for distributed IoT devices, especially in industrial and smart-building scenarios. The project provides a single, seamless foundation for:

- **Authentication & Authorization** — Managing the full authentication lifecycle, token refresh, and role-based access control (admin, engineer, operator).
- **Gateway–Node Orchestration** — Registering, activating, deactivating, and monitoring gateways along with their sensor and actuator nodes.
- **Spatial Management** — Defining managed areas, synchronizing nodes by location, and pathfinding for logistics and movement scenarios.
- **Real-Time Processing** — Ingesting telemetry, heartbeats, and ACKs from ESP32 devices via MQTT; streaming events through SSE & WebSocket; and queuing control commands with full ACK traceability.

The platform aims to reduce the complexity of deploying large-scale IoT systems, enhance observability, and ensure scalability from edge to cloud.

---

## Architecture Overview

The platform follows a layered architecture with a clear separation between the management backend, the operations frontend, and the real-time processing server:

```
┌─────────────────────────────────────────────────────────────┐
│                      Edge / Device Layer                    │
│         ESP32 Gateways  ←→  Sensors / Actuators             │
│         (MQTT Publish: telemetry, heartbeat, ACK)           │
└───────────────────────────┬─────────────────────────────────┘
                            │ MQTT (Aedes :1883)
┌───────────────────────────▼─────────────────────────────────┐
│                   Realtime Server (Node.js)                 │
│  Express API  +  SSE Streams  +  WebSocket  +  Aedes MQTT   │
│  • Telemetry ingestion    • Control queue orchestration     │
│  • ACK tracking           • Device status & whitelist sync  │
└───────────────────────────┬─────────────────────────────────┘
                            │ REST API / Auth (Passport)
┌───────────────────────────▼─────────────────────────────────┐
│                     Backend (Laravel)                       │
│  • Core API: Auth, Users, Company, Notifications, Logs      │
│  • ControlModule: Gateway, Node, Control URL, Workflow      │
│  • MapModule: Managed Area, Node Sync, Route                │
└───────────────────────────┬─────────────────────────────────┘
                            │ REST API
┌───────────────────────────▼─────────────────────────────────┐
│                    Frontend (Nuxt / Vue)                    │
│  Dashboard  ·  Workflow / Scenario Execution  ·  Map Ops    │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Modules

<p align="center">
  <img src="https://github.com/IoTCoreTeam/.github/blob/main/dashboard.png" alt="IoT Core Team dashboard" style="width: 100%; height: auto;">
</p>

### Core API

The central API layer handles authentication, user management, and organization metadata:

- **Auth lifecycle** — `POST /api/login`, `POST /api/refresh-token`, `GET /api/user`, `POST /api/logout`, `POST /api/change-password`.
- **User management** — List, filter, update, delete users, and view role-distribution metrics (for admins and engineers).
- **Company profile** — Read and update organization metadata.
- **Notifications** — List notifications and mark all as read.
- **System logs** — Admin log listing and system metrics endpoints.

### ControlModule

Versioned device management API at `/api/v1/*`, protected by `auth:api` and `admin_or_engineer` middleware:

- **Gateway APIs** — Index, register, deactivate, and delete gateways.
- **Node APIs** — Index, register, and deactivate nodes; public endpoint `GET /api/available-nodes` returns the active-node inventory.
- **Control URL APIs** — CRUD control URLs and execute remote control commands.
- **Control Analog Signal APIs** — List and create-or-update analog signal mappings.
- **Workflow APIs** — CRUD workflows, run / stop execution, and real-time run streams.

### MapModule

<p align="center">
  <img src="https://github.com/IoTCoreTeam/.github/blob/main/workflow.png" alt="IoT Core Team workflow" style="width: 100%; height: auto;">
</p>

Spatial management and routing for campuses and factories:

- **Managed area APIs** — CRUD area definitions.
- **Area–node sync** — Synchronize nodes into areas by external node id.
- **Pathfinding** — Nearest-path lookup endpoint (`RouteController`) for movement and logistics scenarios.

### Realtime Server

<p align="center">
  <img src="https://github.com/IoTCoreTeam/.github/blob/main/control-log.png" alt="IoT Core Team control-log" style="width: 100%; height: auto;">
</p>

The real-time data processing and stream delivery layer:

- **REST API** (`/v1`) — sensors / metrics, whitelist, control queue, device status, ACK analytics, and workflow events.
- **SSE**
  - `/events/gateways` — gateway and node status updates.
  - `/events/control-queue` — control and workflow status streams.
- **WebSocket** — Device state channel (`REQUEST_DEVICE_STATUS` / `DEVICE_STATUS_UPDATE`).
- **MQTT Broker (Aedes :1883)** with standardized topics:
  - `esp32/sensors/data`
  - `esp32/heartbeat`
  - `esp32/nodes/heartbeat`, `esp32/controllers/heartbeat`
  - `esp32/servo/ack`
  - `esp32/control/ack`, `esp32/actuator/ack`
  - `esp32/controllers/status-event`
  - **Whitelist Sync** — `esp32/whitelist/{gatewayId}` (retained) enables automatic re-authorization when a gateway reconnects.

---

## Security & Access Control

- **Laravel Backend** — Bearer token authentication powered by Laravel Passport; APIs gated by `auth:api`, `admin`, and `admin_or_engineer` middleware.
- **Realtime Server** — Service-token middleware protects write-sensitive endpoints (`/v1/control/*`, `/v1/device-status/*`, `/v1/whitelist/*`, `/v1/workflow-events/status`).
- **CORS** — Explicit CORS configuration on the Express API server.
- **Observability** — System log metrics and control ACK query / overview endpoints provide operators with comprehensive visibility into platform health and command delivery.

---

## Tech Stack

<p>
  <img src="https://img.shields.io/badge/Nuxt.js-Frontend-00DC82?style=for-the-badge&logo=nuxtdotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/Vue.js-Framework-42b883?style=for-the-badge&logo=vuedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/Laravel-Backend-FF2D20?style=for-the-badge&logo=laravel&logoColor=white" />
  <img src="https://img.shields.io/badge/Node.js-Express-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-Database-4479A1?style=for-the-badge&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/MongoDB-Database-47A248?style=for-the-badge&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/ESP32-Device-000000?style=for-the-badge&logo=espressif&logoColor=white" />
</p>

---

## Reference

> **Student Research Report SV2526: IoT Device Management and Monitoring System (2025–2026)** — Vietnam Maritime University.  
> [View document](https://www.studocu.vn/vn/document/dai-hoc-hang-hai-viet-nam/nckh2025-2026/bao-cao-nckh-sv2526-he-thong-quan-ly-va-giam-sat-thiet-bi-iot-2025-2026/160234091)

---

*IoT Core Team — Building an open, secure, and scalable IoT foundation.*
