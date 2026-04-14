# 🚀 Blockchain Infra Playbook

A hands-on playbook documenting my journey into blockchain infrastructure engineering through real Linux operations, Ethereum node operation, monitoring, debugging, and observability.

This repository is built as a practical field manual — not just notes — capturing real system behaviour, failures, fixes, and operational workflows while running a live Ethereum node stack on Ubuntu (WSL2).

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
  Prometheus, Node Exporter, Grafana, metrics endpoints, health checks

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

This playbook documents a fully working Ethereum node stack.

### Core Node

- Nethermind running as a systemd service
- Lighthouse running as a systemd service
- JWT-secured Engine API communication (port 8551)
- Execution ↔ consensus interaction verified via logs and forkchoice updates

### Monitoring Stack

- Node Exporter exposing system metrics (port 9100)
- Nethermind metrics exposed (port 6060)
- Prometheus scraping all targets (port 9090)
- Grafana dashboards visualising node + system health (port 3000)

### Validation & Observability

- Prometheus API queries (/api/v1/query, /targets)
- `up` metric verification across all services
- direct metric inspection via curl
- port-level validation using ss
- system health verification via memory, disk, and process checks

### Storage + Performance

- Nethermind data stored on external NVMe mount:

        /mnt/n/nethermind-data

- pruning configured (hybrid mode, volume-based trigger)
- Lighthouse data stored at:

        /var/lib/lighthouse

- WSL2 tuned with custom memory/CPU allocation

---

## 📂 Repository Structure

    blockchain-infra-playbook/
    ├── linux/        → commands, processes, system inspection
    ├── networking/   → ports, RPC, TCP/UDP, connectivity
    ├── systemd/      → service setup, configs, logs
    ├── monitoring/   → Prometheus, Grafana setup
    ├── nodes/        → node ops, architecture, operator checklist
    ├── assets/       → screenshots / proof of system state
    └── README.md

---

## ⚙️ Real Operator Capabilities Demonstrated

This repository demonstrates the ability to:

- run and manage multi-service infrastructure using systemd
- validate system state using logs, ports, and metrics
- debug service interaction issues (execution ↔ consensus)
- inspect memory, disk, and process behaviour
- verify monitoring pipelines end-to-end
- query Prometheus directly via API
- understand and secure internal service communication (JWT)
- operate and monitor a live syncing blockchain node

---

## 📊 Live System Signals

This system is actively validated across multiple layers.

### Service Layer

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager

Confirms services are active and running.

---

### Network Layer

    ss -tulpn | grep -E '8551|6060|9100|9090|3000'

Confirms expected ports:

- 8551 → Engine API (localhost only)
- 6060 → Nethermind metrics
- 9100 → Node Exporter
- 9090 → Prometheus
- 3000 → Grafana

---

### Metrics Layer

Check Prometheus targets:

    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'

Run health query:

    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

Expected result:
- all services return `"up": 1`

---

### Data Layer

- continuous disk growth during sync
- active block processing in Nethermind logs
- Lighthouse peer count and head progression

---

## 📸 Evidence

This repository includes proof-of-life evidence:

- systemctl service status screenshots
- Lighthouse syncing logs
- Nethermind block + forkchoice logs
- Prometheus targets health
- Grafana dashboards
- open ports and system metrics
- disk growth during sync

This demonstrates that the system is not just configured — it is actively running and monitored.

---

## 🔄 What I’m Working On

- strengthening monitoring and alert-style thinking
- improving system debugging speed and confidence
- expanding observability (metrics + logs correlation)
- learning automation and scripting for operations
- refining operator workflows and documentation
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

- WSL2 environment tuned (CPU, memory, disk)
- Nethermind + Lighthouse fully operational
- Engine API secured via JWT
- monitoring stack fully deployed (Prometheus + Grafana)
- metrics scraping verified across all services
- API-level monitoring checks implemented
- operator checklist built for daily/weekly workflows
- real debugging experience across logs, ports, and services

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
