# 🚀 Blockchain Infra Playbook

A hands-on playbook documenting my journey into blockchain infrastructure engineering through real Linux operations, Ethereum node operation, monitoring, debugging, and observability.

This repository is built as a practical field manual, not just notes, capturing real system behaviour, failures, fixes, and operational workflows while running a live Ethereum node stack on Ubuntu via WSL2.

---

## 🧠 What This Repo Is

This repo is a living record of real infrastructure work.

It focuses on learning by doing:

- running and maintaining real services
- validating system health through logs, ports, and metrics
- understanding how execution and consensus clients interact
- debugging real issues across system, network, and protocol layers
- building repeatable operational workflows
- documenting everything as a reusable operator playbook

The goal is to move from:

> **“following tutorials” → “operating systems confidently”**

---

## 🔍 Focus Areas

- 🐧 **Linux fundamentals**  
  Processes, permissions, filesystem, memory, disk, and system inspection

- ⚙️ **systemd**  
  Service lifecycle, startup behaviour, restart logic, logs, and configuration

- 🌐 **Networking**  
  TCP/UDP, ports, localhost vs exposed interfaces, RPC, Engine API, P2P

- ⛓️ **Blockchain infrastructure**  
  Execution + consensus clients, Engine API, JWT auth, syncing, node architecture

- 📊 **Monitoring & Observability**  
  Prometheus, Node Exporter, Grafana, metrics endpoints, health checks, dashboard design

- 🔧 **Debugging workflows**  
  Logs, process inspection, port validation, resource analysis, failure diagnosis

- 📚 **Operational documentation**  
  Turning real system behaviour into structured, reusable playbooks

---

## 🧰 Current Stack

- Windows 11 Pro
- Ubuntu on WSL2
- Linux CLI / Bash
- systemd

### Blockchain Clients
- Nethermind (execution client)
- Lighthouse (consensus client)

### Monitoring Stack
- Prometheus
- Node Exporter
- Grafana

### CLI Tools
- curl, ss, lsof, htop
- journalctl, systemctl
- jq, df, du, free

---

## 🔥 What I’ve Built

This playbook documents a fully working Ethereum node stack with host-level and client-level monitoring.

### Core Node

- Nethermind running as a systemd service
- Lighthouse running as a systemd service
- JWT-secured Engine API communication on port `8551`
- execution ↔ consensus interaction verified via logs and forkchoice updates

### Monitoring Stack

- Node Exporter exposing system metrics on port `9100`
- Nethermind metrics exposed on port `6060`
- Lighthouse metrics exposed and scraped by Prometheus
- Prometheus scraping all targets on port `9090`
- Grafana dashboards visualising host, network, storage, Lighthouse, and Nethermind health on port `3000`

### Validation & Observability

- Prometheus API queries (`/api/v1/query`, `/targets`)
- `up` metric verification across services
- direct metric inspection via `curl`
- port-level validation using `ss`
- system health verification via memory, disk, CPU, load, and process checks
- execution- and consensus-client visibility through Grafana panels

### Storage + Performance

- Nethermind data stored on external NVMe mount:

        /mnt/n/nethermind-data

- pruning configured in hybrid mode with a volume-based trigger
- Lighthouse data stored at:

        /var/lib/lighthouse

- WSL2 tuned with custom memory / CPU allocation
- disk usage and disk I/O now monitored through Grafana

---

## 📂 Repository Structure

    blockchain-infra-playbook/
    ├── linux/        → commands, processes, system inspection
    ├── networking/   → ports, RPC, TCP/UDP, connectivity
    ├── systemd/      → service setup, configs, logs
    ├── monitoring/   → Prometheus, Grafana setup and dashboards
    ├── nodes/        → node ops, architecture, operator checklist
    ├── assets/       → screenshots / proof of system state
    └── README.md

---

## ⚙️ Real Operator Capabilities Demonstrated

This repository demonstrates the ability to:

- run and manage multi-service infrastructure using systemd
- validate system state using logs, ports, and metrics
- debug service interaction issues between execution and consensus clients
- inspect memory, disk, CPU, network, and process behaviour
- verify monitoring pipelines end-to-end
- query Prometheus directly via API
- understand and secure internal service communication with JWT auth
- operate and monitor a live blockchain node across host, consensus, and execution layers

---

## 📊 Monitoring Coverage

The Grafana dashboard now covers multiple operational layers.

### Host Health
- CPU usage
- memory usage
- load average
- system uptime
- Prometheus scrape health
- Node Exporter scrape health

### Network
- network receive throughput
- network transmit throughput

### Storage
- root disk used %
- root free space
- root disk used % over time
- root free space over time
- disk read throughput
- disk write throughput

### Lighthouse
- Lighthouse up
- synced status
- peer count
- current epoch
- finalized epoch
- head slot
- slot lag
- sync slots/sec
- head slot rate

### Nethermind
- Nethermind up
- sync status
- peers
- sync peers
- block lag
- blocks observed (5m)
- chain height
- best known block
- block processing time

This turns the stack from “services installed” into a real operator-facing monitoring view.

---

## 📡 Live System Signals

This system is actively validated across multiple layers.

### Service Layer

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager

Confirms services are active and running.

---

### Network Layer

    ss -tulpn | grep -E '8551|6060|9100|9090|3000'

Confirms expected ports:

- `8551` → Engine API (localhost only)
- `6060` → Nethermind metrics
- `9100` → Node Exporter
- `9090` → Prometheus
- `3000` → Grafana

---

### Metrics Layer

Check Prometheus targets:

    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'

Run health query:

    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

Expected result:

- all configured services return `"up": 1`

---

### Data Layer

- continuous block progression in both execution and consensus views
- active peer connectivity in Lighthouse and Nethermind
- low or zero block lag during healthy operation
- visible disk and network activity during node operation
- block processing visibility through Nethermind metrics

---

## 📸 Evidence

This repository includes proof-of-life evidence such as:

- systemctl service status screenshots
- Lighthouse syncing logs
- Nethermind block and forkchoice logs
- Prometheus target health
- Grafana host-health dashboards
- Grafana Lighthouse monitoring dashboards
- Grafana Nethermind monitoring dashboards
- network, storage, and disk I/O screenshots
- open ports and system metrics
- disk growth and active execution / consensus progress

### Screenshot Examples

- `assets/screenshots/monitoring/Nethermind-and-lighthouse-dashboard-overview.png`
- `assets/screenshots/monitoring/Nethermind-dashboard-overview.png`
- `assets/screenshots/monitoring/lighthouse-dashboard-overview.png`
- `assets/screenshots/monitoring/grafana-host-health-dashboard.png`
- `assets/screenshots/monitoring/grafana-network-dashboard.png`
- `assets/screenshots/monitoring/grafana-storage-dashboard.png`

This demonstrates that the system is not just configured, it is actively running, monitored, and documented.

---

## 🧪 Key Things Learned So Far

- `up = 1` means Prometheus can successfully scrape that target
- execution and consensus monitoring expose different health signals and both matter
- Lighthouse sync is better measured through real sync metrics than visual guessing
- Nethermind metrics often require `sum()` or `max()` to turn raw series into clean panels
- block lag is a simple and useful execution-client signal:

        best known block - local chain height

- lower block processing time is better because it reflects faster block handling
- disk I/O matters because node performance can be limited by storage behaviour even when CPU looks fine
- grouping dashboards by host, network, storage, consensus, and execution makes troubleshooting much faster
- WSL-mounted filesystems behave differently from native Linux filesystems, so reliable monitoring should focus on the native Linux root filesystem where appropriate

---

## 🔄 What I’m Working On

- strengthening monitoring and alert-style thinking
- improving debugging speed and confidence
- expanding observability through logs + metrics correlation
- learning automation and scripting for operations
- refining operator workflows and documentation
- practicing service recovery and validation workflows
- preparing for real-world infra / SRE environments

---

## 🎯 Goal

To become a high-level infrastructure engineer capable of:

- operating blockchain nodes reliably
- debugging distributed systems
- understanding execution + consensus deeply
- building observable, production-style systems
- documenting infrastructure clearly and professionally

---

## 📈 Current Progress Snapshot

- WSL2 environment tuned for node workloads
- Nethermind + Lighthouse fully operational
- Engine API secured via JWT
- Prometheus + Grafana fully deployed
- Node Exporter scraping verified
- Nethermind metrics scraping verified
- Lighthouse metrics scraping verified
- Grafana dashboards built for host, network, storage, Lighthouse, and Nethermind
- API-level monitoring checks implemented
- operator checklist built for daily and weekly workflows
- real debugging experience across logs, ports, JWT permissions, scraping, and service interaction

---

## 🧭 Journey Timeline

- Started infrastructure journey: April 9, 2026
- First full Ethereum node + monitoring stack operational: April 12, 2026
- Expanded monitoring to include host, Lighthouse, Nethermind, storage, and disk I/O dashboards: April 14, 2026

---

## 🧭 Philosophy

This repository is intentionally practical.

It documents:

- what I ran
- what broke
- how I diagnosed it
- how I fixed it
- what I learned

The focus is not just knowledge, but:

> **building real operational confidence through hands-on systems**

---

## 🚀 End State

To think and operate like an infrastructure engineer:

- observe before acting
- verify before assuming
- debug methodically
- document clearly
- build systems that are understandable and reliable
