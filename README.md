# 🚀 Blockchain Infra Playbook

A hands-on playbook documenting my journey into blockchain infrastructure engineering through real Linux operations, Ethereum node setup, service management, networking, debugging, and observability.

This repository is built as a practical field manual, not just a study notebook. It captures the commands I use, the systems I run, the issues I hit, and the lessons I learn while building real blockchain infrastructure on Ubuntu WSL2.

---

## 🧠 Focus Areas

- 🐧 Linux fundamentals: commands, processes, permissions, filesystem awareness
- ⚙️ `systemd`: services, unit files, startup behavior, log inspection
- 🌐 Networking: TCP/UDP, ports, localhost vs public interfaces, RPC concepts
- ⛓️ Blockchain nodes: execution clients, consensus clients, Engine API, syncing
- 🔍 Debugging and monitoring: logs, processes, ports, disk growth, health checks
- 🛠️ Real-world operations: installing, verifying, restarting, migrating, documenting

---

## 🧰 Current Stack

- Windows 11 Pro
- Ubuntu on WSL2
- Linux CLI / Bash
- `systemd`
- Nethermind (execution client)
- Lighthouse (consensus client)
- CLI tools: `curl`, `wget`, `ss`, `lsof`, `htop`, `jq`, `journalctl`, `systemctl`

---

## 📂 Repository Structure

    linux/      → Linux commands, foundations, process/resource checks
    networking/ → ports, TCP/UDP, RPC, connectivity notes
    systemd/    → service setup, service management, logs, configs
    nodes/      → node architecture, execution vs consensus notes
    assets/     → screenshots and supporting evidence
    README.md   → root overview of the playbook

---

## 🛠️ Node Operations

### Nethermind

- [Nethermind Install and Service Setup](systemd/nethermind-install-and-service-setup.md)
- [Nethermind Operator Checks](systemd/nethermind-operator-checks.md)
- [Nethermind Screenshot Evidence](assets/screenshots/nethermind/README.md)

These documents cover:
- installing Nethermind on Ubuntu WSL2
- configuring Nethermind as a `systemd` service
- checking logs, ports, disk usage, and process health
- documenting proof-of-life screenshots from the live environment

### Consensus Client Notes

- [Consensus Client Notes](systemd/consensus-client-notes.md)

This document covers:
- what a consensus client is
- why Nethermind alone waits for forkchoice
- what port `8551` is used for
- why Lighthouse is the chosen consensus client for this setup

### Lighthouse

- [Lighthouse Install and Setup](systemd/lighthouse-install-and-setup.md)

This document covers:
- creating a shared JWT secret
- connecting Lighthouse to Nethermind through the Engine API
- running Lighthouse as a `systemd` service
- verifying execution and consensus client communication

### Full Ethereum Node Evidence

- [Nethermind + Lighthouse Screenshot Evidence](assets/screenshots/eth-node-nethermind-lighthouse/README.md)

This folder contains screenshot evidence for:
- Nethermind and Lighthouse service status
- Lighthouse syncing logs
- Nethermind receiving forkchoice updates
- open ports
- disk and chain-data growth

---

## 🔥 What I'm Working On

- Running and documenting a full Ethereum node stack
- Managing Nethermind and Lighthouse through `systemd`
- Understanding execution layer vs consensus layer responsibilities
- Monitoring service health through logs, ports, and storage growth
- Building a portfolio of real infrastructure notes, evidence, and operational workflows

---

## 🎯 Goal

To become a high-level blockchain infrastructure engineer capable of:

- running and maintaining blockchain nodes reliably
- debugging distributed systems and service interactions
- understanding execution and consensus client architecture
- building observable, dependable, real-world infrastructure
- documenting systems clearly enough for others to learn from and trust

---

## 📌 Philosophy of This Repo

This repository is intentionally practical.

It is not just a collection of theoretical notes. It is a running log of:
- what I installed
- how I configured it
- how I verified it
- what I observed
- what I learned
- how I would explain it to someone coming after me

The goal is to grow from “following setup steps” into “thinking and operating like an infrastructure engineer.”

---

## 📈 Current Progress Snapshot

So far, this playbook includes:

- Ubuntu WSL2 environment setup
- Nethermind installed and managed as a `systemd` service
- service inspection with `systemctl`
- log inspection with `journalctl`
- process, port, and disk checks
- WSL distro migration to a larger NVMe-backed location
- consensus-client architecture notes
- Lighthouse setup documentation
- screenshot evidence for both execution-only and execution+consensus stages

---

## 📌 Notes

This repo is continuously updated as I learn by building and operating real systems.

Each file is meant to be a practical artifact from hands-on work, with enough clarity that I can revisit it later and understand not just what I did, but why I did it.
