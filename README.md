# 🚀 Blockchain Infra Playbook

A hands-on playbook documenting my journey into blockchain infrastructure engineering through real Linux operations, Ethereum node setup, service management, networking, debugging, and observability.

This repository is built as a practical field manual, not just a study notebook. It captures the commands I use, the systems I run, the issues I hit, the fixes I make, and the lessons I learn while building real blockchain infrastructure on Ubuntu WSL2.

---

## 🧠 What This Repo Is

This repo is a living record of real infrastructure work.

It focuses on learning by doing:
- installing and operating services
- reading logs and validating system state
- understanding how blockchain node components fit together
- documenting real troubleshooting and operational workflows
- building the habits of an infrastructure engineer through repetition and evidence

The goal is not just to collect notes, but to build a reusable operator playbook.

---

## 🔍 Focus Areas

- 🐧 **Linux fundamentals**  
  Commands, processes, permissions, filesystem awareness, and service users

- ⚙️ **systemd**  
  Unit files, service startup, boot behavior, restart handling, and log inspection

- 🌐 **Networking**  
  TCP/UDP, ports, localhost vs public interfaces, peer-to-peer networking, and RPC concepts

- ⛓️ **Blockchain nodes**  
  Execution clients, consensus clients, Engine API, JWT authentication, syncing, and node architecture

- 🔧 **Debugging and monitoring**  
  Logs, process checks, port verification, disk growth, health checks, and sync-state reasoning

- 📚 **Operational documentation**  
  Turning hands-on setup, debugging, and maintenance into clear written playbooks

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

## 🔥 What I’ve Built

So far, this playbook documents a working local Ethereum node stack with:

- Nethermind running as a `systemd` service
- Lighthouse running as a `systemd` service
- JWT-authenticated Engine API communication between execution and consensus clients
- active sync monitoring through logs, ports, and storage growth
- screenshot evidence showing live service health and execution-consensus interaction
- written notes covering installation, debugging, verification, and daily operator workflows

---

## 📂 Repository Structure

    linux/      → Linux commands, foundations, process/resource checks
    networking/ → Ports, TCP/UDP, RPC, connectivity notes
    systemd/    → Service setup, management, logs, configs, and client install notes
    nodes/      → Node architecture, operator workflows, and execution vs consensus notes
    assets/     → Screenshots and supporting evidence
    README.md   → Root overview of the playbook

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

---

### Consensus Client Notes

- [Consensus Client Notes](systemd/consensus-client-notes.md)

This document covers:
- what a consensus client is
- why Nethermind alone waits for forkchoice
- what port `8551` is used for
- why Lighthouse is the chosen consensus client for this setup

---

### Lighthouse

- [Lighthouse Install and Setup](systemd/lighthouse-install-and-setup.md)

This document covers:
- downloading and installing Lighthouse
- sharing JWT access securely between service users
- connecting Lighthouse to Nethermind through the Engine API
- manually testing Lighthouse before automation
- running Lighthouse as a `systemd` service
- verifying execution and consensus client communication

---

### Full Ethereum Node Evidence

- [Nethermind + Lighthouse Screenshot Evidence](assets/screenshots/eth-node-nethermind-lighthouse/README.md)

This folder contains screenshot evidence for:
- Nethermind and Lighthouse service status
- Lighthouse syncing logs
- Nethermind receiving forkchoice updates
- open ports
- disk and chain-data growth

---

## 📸 Live Node Status

This repository includes proof-of-life evidence from a live Ethereum node environment, including:

- execution client running under `systemd`
- consensus client running under `systemd`
- successful forkchoice updates from Lighthouse to Nethermind
- active listening ports for execution, Engine API, and consensus networking
- storage growth from live chain sync

This evidence is included not just to show that the setup exists, but to demonstrate that it is functioning and being actively operated.

---

## 🔄 What I’m Working On

Right now, I’m focused on:

- operating and documenting a full Ethereum node stack
- strengthening Linux and `systemd` fundamentals through real use
- understanding execution vs consensus responsibilities more deeply
- improving node health checks, log reading, and sync-state interpretation
- turning hands-on setup and debugging into reusable documentation
- building a portfolio of practical blockchain infrastructure work

---

## 🎯 Goal

To become a high-level blockchain infrastructure engineer capable of:

- running and maintaining blockchain nodes reliably
- debugging distributed systems and service interactions
- understanding execution and consensus client architecture deeply
- building observable, dependable, real-world infrastructure
- documenting systems clearly enough for other operators to learn from and trust

---

## 📌 Philosophy of This Repo

This repository is intentionally practical.

It is not just a collection of theoretical notes. It is a running log of:
- what I installed
- how I configured it
- how I verified it
- what issues I ran into
- how I fixed them
- what I learned from the process
- how I would explain it to someone coming after me

The aim is to move from “following setup steps” to “thinking and operating like an infrastructure engineer.”

---

## 📈 Current Progress Snapshot

This playbook currently includes:

- Ubuntu WSL2 environment setup
- Nethermind installed and managed as a `systemd` service
- service inspection with `systemctl`
- log inspection with `journalctl`
- process, port, and disk checks
- WSL distro migration to a larger NVMe-backed location
- consensus-client architecture notes
- Lighthouse setup documentation
- full execution + consensus node evidence
- screenshot evidence for both execution-only and full-node stages
- operator checklists for daily and weekly node monitoring

---

## 🧭 Why This Repo Exists

I’m using this repository to build real operational confidence.

That means learning how to:
- inspect systems instead of guessing
- verify services instead of assuming
- understand logs instead of ignoring them
- document infrastructure in a way that is both technically useful and repeatable

Over time, this playbook is meant to grow into a solid record of practical blockchain infrastructure capability.

---

## 📌 Notes

This repo is continuously updated as I learn by building and operating real systems.

Each file is meant to be a practical artifact from hands-on work, with enough clarity that I can revisit it later and understand not just what I did, but why I did it.
