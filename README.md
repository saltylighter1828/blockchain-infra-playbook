# 🚀 Blockchain Infra Playbook

A hands-on infrastructure portfolio focused on Ethereum node operations, Linux systems work, observability, and service reliability.

This repository documents the design, operation, monitoring, and troubleshooting of a live Ethereum node stack running on Ubuntu via WSL2, using Nethermind, Lighthouse, Prometheus, Grafana, and Node Exporter. It is built as an operator-facing playbook that captures real system behaviour, real failure modes, and the workflows used to validate and recover services.

---

## Overview

This project is designed around practical infrastructure work rather than theory alone.

It focuses on:

- operating real services with `systemd`
- understanding execution and consensus client interaction
- validating system health through logs, ports, metrics, and APIs
- building observability across host, execution, and consensus layers
- documenting repeatable operational workflows
- turning manual node operations into structured runbooks and Bash tooling

The broader goal is to develop production-style infrastructure habits through direct hands-on operation.

---

## Technical Focus

### Linux & Systems
- process inspection
- filesystem and storage awareness
- memory and disk analysis
- service lifecycle management
- command-line operations in Ubuntu on WSL2

### Service Operations
- `systemd` unit management
- startup and restart sequencing
- service validation
- live and historical log inspection
- controlled shutdown and recovery workflows

### Networking
- localhost-bound service design
- TCP/UDP port inspection
- RPC and Engine API validation
- metrics endpoint verification
- internal service communication patterns

### Blockchain Infrastructure
- Nethermind as the execution client
- Lighthouse as the consensus client
- Engine API connectivity via JWT authentication
- sync verification and peer visibility
- execution and consensus observability

### Monitoring & Observability
- Prometheus scrape configuration and validation
- Grafana dashboard design
- Node Exporter host metrics
- direct Prometheus API verification
- metrics-based health checks for both clients

### Documentation & Automation
- operational checklists
- recovery runbooks
- Bash-based validation tooling
- repeatable workflows for node inspection and control

---

## Current Stack

### Platform
- Windows 11 Pro
- Ubuntu on WSL2
- Bash / Linux CLI
- `systemd`

### Blockchain Clients
- Nethermind
- Lighthouse

### Monitoring Stack
- Prometheus
- Node Exporter
- Grafana

### Common Tools
- `curl`
- `ss`
- `lsof`
- `htop`
- `journalctl`
- `systemctl`
- `jq`
- `df`
- `du`
- `free`

---

## What This Repository Demonstrates

This repository demonstrates practical capability in:

- running and managing a multi-service blockchain infrastructure stack
- operating execution and consensus clients together with correct service dependency flow
- securing execution-consensus communication using JWT-authenticated Engine API access
- validating services using system state, ports, logs, APIs, and metrics
- building observability across host, consensus, and execution layers
- debugging service interaction, scrape health, endpoint readiness, and startup timing issues
- documenting real operational behaviour in a structured and reusable way

---

## Implemented Node Stack

### Core Node
- Nethermind running as a `systemd` service
- Lighthouse running as a `systemd` service
- Engine API communication secured with JWT on port `8551`
- execution ↔ consensus interaction verified through logs and forkchoice updates

### Monitoring
- Node Exporter exposing host metrics on port `9100`
- Nethermind metrics exposed on port `6060`
- Lighthouse metrics exposed on port `5054`
- Prometheus scraping configured targets on port `9090`
- Grafana visualising host, network, storage, Lighthouse, and Nethermind health on port `3000`

### Validation
- Prometheus API verification via `/targets` and `/api/v1/query`
- `up` metric confirmation across configured scrape jobs
- direct metrics inspection with `curl`
- port-level verification using `ss`
- system resource validation through memory, disk, load, uptime, and process checks
- sync and peer visibility through both raw logs and dashboard panels

### Storage & Runtime
- Nethermind data stored on external NVMe at:

        /mnt/n/nethermind-data

- Nethermind pruning configured in hybrid mode with volume-based trigger
- Lighthouse data stored at:

        /var/lib/lighthouse

- WSL2 configured with tuned memory / CPU allocation for node workloads

---

## Monitoring Coverage

Grafana dashboards currently cover multiple operational layers.

### Host Health
- CPU usage
- memory usage
- load average
- system uptime
- Prometheus scrape health
- Node Exporter scrape health

### Network
- receive throughput
- transmit throughput

### Storage
- root disk used %
- root free space
- disk usage trends
- read throughput
- write throughput

### Lighthouse
- service availability
- synced status
- peer count
- current epoch
- finalized epoch
- head slot
- slot lag
- sync slots per second
- head slot growth rate

### Nethermind
- service availability
- sync status
- peer count
- sync peers
- block lag
- blocks observed
- chain height
- best known block
- block processing time

This turns the stack from a basic deployment into an operator-facing monitored environment.

---

## Operational Validation

The system is validated across several layers.

### Service Layer

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager

Used to confirm service activity and runtime state.

### Network Layer

    ss -tulpn | grep -E '8551|6060|9100|9090|3000'

Used to confirm the expected local ports are listening.

Expected ports include:

- `8551` → Engine API
- `6060` → Nethermind metrics
- `9100` → Node Exporter
- `9090` → Prometheus
- `3000` → Grafana

### Metrics Layer

    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

Used to confirm Prometheus target health and scrape success.

### Data Layer
- continuous execution and consensus progression
- active peer connectivity
- low or zero block lag during healthy operation
- visible disk and network activity
- observable block processing behaviour

---

## Bash Tooling

This repository also includes an operational scripts layer for repeatable node workflows.

Current tooling includes:

- `node-check.sh`
- `service-validate.sh`
- `quick-logs.sh`
- `live-logs.sh`
- `startup-services.sh`
- `shutdown-services.sh`
- `restart-services.sh`
- `shutdown-and-exit-wsl.sh`
- `metrics-check.sh`

These scripts package common checks and recovery actions into reusable operator utilities, covering:

- health snapshots
- service validation
- log inspection
- startup / shutdown / restart flows
- monitoring and metrics verification

---

## Repository Structure

    blockchain-infra-playbook/
    ├── linux/        → commands, processes, system inspection
    ├── networking/   → ports, RPC, TCP/UDP, connectivity
    ├── systemd/      → service setup, configs, logs
    ├── monitoring/   → Prometheus, Grafana setup and dashboards
    ├── nodes/        → node operations, architecture, operator checklists
    ├── scripts/      → Bash tooling for validation and recovery workflows
    ├── assets/       → screenshots and supporting evidence
    └── README.md

---

## Evidence

This repository includes proof-of-life evidence from the running stack, including:

- `systemctl` service status output
- Lighthouse sync logs
- Nethermind block and forkchoice logs
- Prometheus target health
- Grafana host-health dashboards
- Grafana Lighthouse dashboards
- Grafana Nethermind dashboards
- network, storage, and disk I/O screenshots
- open port validation
- live execution and consensus progress

Example screenshots include:

- `assets/screenshots/monitoring/Nethermind-and-lighthouse-dashboard-overview.png`
- `assets/screenshots/monitoring/Nethermind-dashboard-overview.png`
- `assets/screenshots/monitoring/lighthouse-dashboard-overview.png`
- `assets/screenshots/monitoring/grafana-host-health-dashboard.png`
- `assets/screenshots/monitoring/grafana-network-dashboard.png`
- `assets/screenshots/monitoring/grafana-storage-dashboard.png`

These artifacts are included to show that the stack is not only configured, but actively running, monitored, and verified.

---

## Key Practical Learnings

Some of the main operational takeaways from this project so far:

- `up = 1` in Prometheus confirms a successful scrape, not just a running service
- execution and consensus clients expose different health signals and both must be observed
- Lighthouse sync is better measured through metrics than by visual guesswork
- Nethermind metrics often require aggregation functions such as `sum()` or `max()` for useful dashboard panels
- block lag is a simple and effective execution-client health indicator:

        best known block - local chain height

- block processing time is a useful performance signal for execution health
- disk behaviour matters for node performance, even when CPU usage appears normal
- grouping observability by host, network, storage, consensus, and execution speeds up troubleshooting
- WSL-mounted filesystems behave differently from native Linux filesystems, which affects how storage should be interpreted and monitored

---

## Project Status

### Completed
- WSL2 environment tuned for node workloads
- Nethermind + Lighthouse operational
- Engine API secured with JWT
- Prometheus + Grafana deployed
- Node Exporter scraping verified
- Nethermind metrics scraping verified
- Lighthouse metrics scraping verified
- dashboards built for host, network, storage, Lighthouse, and Nethermind
- Prometheus API-level checks implemented
- operational Bash tooling implemented for validation and recovery
- real debugging performed across logs, ports, JWT permissions, scrape health, and service readiness

### In Progress
- strengthening monitoring and alert-style thinking
- improving debugging speed and confidence
- refining infrastructure documentation
- expanding operational automation
- preparing for more production-style infra / SRE workflows

---

## Timeline

- Infrastructure journey started: April 9, 2026
- First full Ethereum node + monitoring stack operational: April 12, 2026
- Monitoring expanded across host, Lighthouse, Nethermind, storage, and disk I/O: April 14, 2026

---

## Career Direction

This project is part of a broader move toward infrastructure engineering, with emphasis on:

- blockchain node operations
- Linux systems work
- monitoring and observability
- debugging distributed systems
- service reliability
- operator-focused automation

The aim is to continue developing toward roles in blockchain infrastructure, platform engineering, SRE, or reliability-focused systems work.

---

## Philosophy

This repository is intentionally practical.

It records:

- what was deployed
- what failed
- how issues were diagnosed
- how recovery was verified
- what was learned from operating the system directly

The focus is not only on understanding commands, but on building operational confidence through repeated, observable, real-world infrastructure work.

---

## End Goal

To operate like an infrastructure engineer who can:

- observe before acting
- verify before assuming
- debug methodically
- document clearly
- build systems that are understandable, monitored, and reliable
