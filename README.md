<div align="center">
  <img src="https://img.shields.io/badge/Docker_Compose-One_Command_Setup-2496ED?logo=docker&logoColor=white" alt="Docker Compose">
  <img src="https://img.shields.io/badge/Temporal.io-Durable_Execution-000?logo=temporal&logoColor=white" alt="Temporal.io">
  <img src="https://img.shields.io/badge/Apache_Kafka-Event_Streaming-231F20?logo=apachekafka&logoColor=white" alt="Kafka">
  <img src="https://img.shields.io/badge/MinIO-Object_Storage-C72E49?logo=minio&logoColor=white" alt="MinIO">
</div>

<h1 align="center">Continuum Local Setup</h1>

<div align="center">
  <strong>The entire Continuum platform. One command. Your machine.</strong><br/>
  Everything you need to build, run, and explore visual workflows — locally.
</div>

---

## What Is This

This repo gives you a single `docker compose up` that launches the **complete** Project Continuum stack — infrastructure services, platform backend, browser IDE, and feature workers — all wired together and ready to go.

No builds. No cloning five repos. Just Docker.

---

## Prerequisites

| Requirement | Minimum | Check |
|-------------|---------|-------|
| **Docker** | 20.10+ | `docker --version` |
| **Docker Compose** | v2.0+ | `docker compose version` |
| **RAM** | 8 GB available for containers | — |
| **Disk** | ~5 GB for images + volumes | — |

> Make sure Docker Desktop (or your Docker daemon) is running before you start.

---

## Quick Start

```bash
cd docker
docker compose up -d
```

That's it. Wait for all containers to become healthy, then open:

**[http://localhost:3002](http://localhost:3002)** — Continuum Workbench (browser IDE)

You'll find example workflows pre-loaded in the IDE under `/home/node/example-workflows/`.

### Verify Everything Is Running

```bash
docker compose ps
```

All services should show `running` (the `temporal-search-attributes-init` container will exit after setup — that's expected).

### Stop Everything

```bash
docker compose down
```

To also remove all stored data (volumes):

```bash
docker compose down -v
```

---

## What's Inside

The compose stack has three layers: **infrastructure**, **platform services**, and **feature workers**.

### Infrastructure Services

These are the open-source foundations that Continuum runs on top of.

| Service | Port | Purpose |
|---------|------|---------|
| **PostgreSQL** | 35432 | Database backing Temporal's workflow state and history |
| **Temporal** | 7233 | Durable workflow execution engine — orchestrates node-by-node execution, handles retries and crash recovery |
| **Temporal UI** | [localhost:38081](http://localhost:38081) | Web dashboard for inspecting running workflows, viewing execution history, and debugging |
| **Temporal Admin Tools** | — | CLI container for managing Temporal namespaces and search attributes (also runs one-time init on startup) |
| **Kafka** (3-node KRaft cluster) | 39092, 39093, 39094 | Distributed event streaming — carries real-time node execution events from workers to the browser |
| **Schema Registry** | [localhost:38080](http://localhost:38080) | Manages Avro schemas for Kafka messages so producers and consumers agree on data format |
| **Kafka UI** | [localhost:38082](http://localhost:38082) | Web dashboard for browsing Kafka topics, messages, and consumer groups |
| **Mosquitto** | 31883 (TCP), 31884 (WebSocket) | Lightweight MQTT broker — the last hop that pushes execution events to the browser over WebSockets |
| **MinIO** | [localhost:39000](http://localhost:39000) (API), [localhost:39001](http://localhost:39001) (Console) | S3-compatible object storage — stores Parquet data files passed between workflow nodes |

### Platform Services

These are the Continuum-specific backend services.

| Service | Port | Purpose |
|---------|------|---------|
| **continuum-api-server** | [localhost:8080](http://localhost:8080) | REST API — manages workflows, the node registry, and triggers executions. The single entry point for the workbench UI |
| **continuum-message-bridge** | — | Kafka-to-MQTT bridge — consumes node execution events from Kafka and republishes them to Mosquitto so the browser gets live updates |
| **continuum-workbench** | [localhost:3002](http://localhost:3002) | Browser IDE — Eclipse Theia + React Flow canvas where you build, configure, and run workflows |

### Feature Workers

Workers are independent services that provide workflow nodes. Each worker registers its nodes with Temporal and waits for activities.

| Service | Nodes | Purpose |
|---------|-------|---------|
| **continuum-feature-base** | 16 nodes | Core analytics — Create Table, Column Join, Pivot, Row Filter, Conditional Splitter, REST Client, Anomaly Detector, Kotlin Script, and more |
| **continuum-feature-cheminformatics** | RDKit nodes | Chemistry — molecular descriptors and cheminformatics operations via RDKit |

> Want AI nodes? Add the `continuum-feature-ai` worker image to the compose file to get LLM fine-tuning with Unsloth.

---

## Example Workflows

The `docker/example-workflows/` directory is mounted into the workbench container. Open any `.cwf` file from the IDE to explore:

| Workflow | What It Demonstrates |
|----------|---------------------|
| `SensorDataPipeline.cwf` | IoT sensor data → anomaly detection → conditional routing → batch alerts via REST |
| `AnomalyDetector.cwf` | Z-score anomaly detection with conditional splitting |
| `RESTNode.cwf` | Making HTTP requests to external APIs from a workflow |
| `Unsloth.cwf` | LLM fine-tuning configuration (requires the AI worker) |

---

## How the Pieces Fit Together

```
┌─────────────────────────────────────────────────────┐
│              BROWSER  (localhost:3002)                │
│   Eclipse Theia IDE + React Flow drag-and-drop       │
└──────────────┬────────────────────┬─────────────────┘
               │ REST               │ MQTT / WebSocket
               ▼                    ▼
      ┌────────────────┐   ┌──────────────────┐
      │  API Server    │   │  Message Bridge   │
      │  (port 8080)   │   │  Kafka → MQTT     │
      └───────┬────────┘   └────────▲─────────┘
              │                     │
              ▼                     │
      ┌──────────────┐    ┌────────┴────────┐
      │   Temporal    │    │     Kafka       │
      │   (port 7233) │    │  (3-node KRaft) │
      └───────┬───────┘    └────────▲────────┘
              │  dispatches          │  events
              ▼                     │
      ┌──────────────────────────────┐
      │         WORKERS              │
      │  Base · Cheminformatics · …  │
      └──────────────┬───────────────┘
                     │ read / write
                     ▼
            ┌─────────────────┐
            │  MinIO (S3)     │
            │  Parquet files  │
            └─────────────────┘
```

1. You design a workflow in the browser and hit **Execute**
2. The API server creates a Temporal workflow
3. Temporal dispatches each node as an activity to the correct worker
4. The worker downloads input data (Parquet) from MinIO, runs the node logic, uploads results back
5. The worker publishes progress events to Kafka
6. The message bridge forwards events via MQTT to the browser
7. You see each node light up in real time

---

## Useful Endpoints

| URL | What |
|-----|------|
| [localhost:3002](http://localhost:3002) | Continuum Workbench (IDE) |
| [localhost:8080](http://localhost:8080) | API Server |
| [localhost:38081](http://localhost:38081) | Temporal UI |
| [localhost:38082](http://localhost:38082) | Kafka UI |
| [localhost:39001](http://localhost:39001) | MinIO Console (user: `minioadmin` / pass: `minioadmin`) |

---

## Troubleshooting

**Containers keep restarting?**
Check logs: `docker compose logs <service-name> --tail 100`

**Temporal not starting?**
PostgreSQL needs to be healthy first. Give it 30–60 seconds on first run.

**Workbench shows connection errors?**
The API server may still be starting. Wait until `docker compose ps` shows all services as `running`.

**Out of memory?**
The Kafka cluster + Temporal + all services need ~6–8 GB. Increase Docker's memory limit in Docker Desktop settings.

---

## Related Repositories

| Repository | Description |
|-----------|-------------|
| [Continuum](https://github.com/projectcontinuum/Continuum) | Core backend — API server, worker framework, shared libraries |
| [continuum-workbench](https://github.com/projectcontinuum/continuum-workbench) | Browser IDE — Eclipse Theia + React Flow workflow editor |
| [continuum-feature-base](https://github.com/projectcontinuum/continuum-feature-base) | Base analytics nodes — transforms, REST, scripting, anomaly detection |
| [continuum-feature-ai](https://github.com/projectcontinuum/continuum-feature-ai) | AI/ML nodes — LLM fine-tuning with Unsloth + LoRA |
| [continuum-feature-cheminformatics](https://github.com/projectcontinuum/continuum-feature-cheminformatics) | Chemistry nodes — RDKit molecular operations |
| [continuum-feature-template](https://github.com/projectcontinuum/continuum-feature-template) | Template — scaffold your own custom worker with nodes |

---

## License

[Apache 2.0](../LICENSE) — open, safe, patent-protected.

---

<div align="center">
  <strong>One command. Full platform. Start building workflows.</strong>
</div>
