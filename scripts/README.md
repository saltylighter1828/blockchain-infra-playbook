# Scripts

This directory contains the Bash tooling used to support day-to-day Ethereum node operations across the local Nethermind, Lighthouse, Prometheus, and Grafana stack.

The purpose of these scripts is to make routine operational tasks more consistent, faster to execute, and easier to verify. Rather than relying on scattered one-off commands, this layer packages common checks and service actions into small, reusable utilities.

These scripts are intended to support practical node operations such as:

- health and status checks
- service validation
- log inspection
- startup, shutdown, and restart workflows
- monitoring and metrics verification

Together, they form a lightweight operator toolkit built around the stack running in this repository.

---

## Overview

Operating a node stack quickly leads to repeated use of the same core system commands:

- `systemctl`
- `journalctl`
- `ss`
- `curl`
- `df`
- `free`
- JSON-RPC queries
- Prometheus API queries

Those commands are useful on their own, but scripting them creates a more reliable operational workflow. The result is better repeatability, clearer verification, and a smoother path from manual experimentation to disciplined infrastructure practice.

This directory reflects that progression: from individual commands to reusable operator tools.

---

## Scope

The scripts in this folder focus on five operational areas:

### 1. Node Health Checks
Quick, single-run checks for service state, ports, system resources, and general node condition.

### 2. Service Validation
Targeted validation of individual services, including status, recent activity, and expected runtime behaviour.

### 3. Log Inspection
Shortcuts for reviewing recent logs or following live output from key services during sync, recovery, or troubleshooting.

### 4. Recovery Workflows
Controlled startup, shutdown, and restart procedures for the execution and consensus clients, with verification steps included.

### 5. Monitoring Verification
Checks that confirm the observability layer is functioning end to end, including Prometheus target health and key node metrics.

---

## Script Inventory

### `node-check.sh`
Provides a fast operational snapshot of the local stack.

Typical checks include:

- service activity
- expected listening ports
- disk usage
- memory usage
- system uptime

This is the quickest way to confirm that the node environment is broadly healthy.

### `service-validate.sh`
Performs deeper service-level validation for the main stack components.

This includes:

- `systemctl is-active`
- service status inspection
- recent journal output

Useful when a service appears active but needs closer verification.

### `quick-logs.sh`
Displays recent logs for a selected service.

Designed for fast inspection of recent behaviour without switching to longer-form log browsing.

### `live-logs.sh`
Follows live logs for a selected service.

Useful during restart testing, sync observation, and active troubleshooting.

### `startup-services.sh`
Starts the execution and consensus clients in the intended order and verifies recovery.

Includes service checks and expected port validation.

### `shutdown-services.sh`
Stops the node services cleanly and confirms they are no longer active.

Useful when pausing node activity without shutting down the entire WSL environment.

### `restart-services.sh`
Restarts the core node services and verifies that the stack recovers cleanly.

Includes service checks, port checks, and monitoring checks.

### `shutdown-and-exit-wsl.sh`
Stops node services cleanly and then shuts down the WSL environment.

Useful when reclaiming local system resources, such as before gaming or other high-load desktop use.

### `metrics-check.sh`
Validates the monitoring chain and confirms that key metrics remain queryable.

Current checks include:

- Prometheus reachability
- scrape target health
- `up` status for expected jobs
- Nethermind JSON-RPC block height
- peer count
- sync status
- Lighthouse sync state
- head slot and slot lag

This script verifies not only that monitoring services are running, but that they are returning meaningful operational data.

---

## Design Approach

This tooling is intentionally lightweight.

The goal is not to build a large automation framework, but to create a practical Bash-based control layer around common node operations. Each script is designed to be:

- readable
- small in scope
- directly runnable
- easy to validate manually
- close to the underlying Linux and node commands

Bash is a natural fit for this stage of the project because it keeps the automation close to the system itself. It also provides a strong foundation before moving into heavier tooling such as Python, Ansible, or Terraform.

---

## Operational Model

These scripts wrap and standardize commands commonly used during node operation, including:

### Service management

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager
    systemctl status prometheus-node-exporter --no-pager

### Log inspection

    journalctl -u nethermind -n 50 --no-pager
    journalctl -u lighthouse-beacon -n 50 --no-pager
    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f

### Port verification

    ss -tulpn | grep -E '8551|6060|5054|9100|9090|3000'

### Resource checks

    df -h /
    free -h
    uptime

### Monitoring and metrics queries

    curl -s http://127.0.0.1:9090/-/healthy
    curl -s http://127.0.0.1:9090/api/v1/targets
    curl -s http://127.0.0.1:9090/api/v1/query?query=up

### Nethermind JSON-RPC

    curl -s -H "Content-Type: application/json" \
      -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
      http://127.0.0.1:8545

---

## Environment

These scripts are currently designed around the local environment used in this repository:

- Ubuntu on WSL2
- `systemd`-managed services
- Nethermind as the execution client
- Lighthouse as the consensus client
- Prometheus for metrics collection
- Grafana for dashboards
- node exporter for host-level metrics

Service names, ports, and endpoints are based on the current local stack and may need adjustment if the environment changes.

---

## Usage

Make scripts executable once:

    chmod +x *.sh

Run them directly from this folder, for example:

    ./node-check.sh
    ./service-validate.sh
    ./quick-logs.sh nethermind
    ./live-logs.sh lighthouse-beacon
    ./startup-services.sh
    ./shutdown-services.sh
    ./restart-services.sh
    ./metrics-check.sh

Some scripts require `sudo` because they interact with `systemd` services.

---

## Why This Matters

This directory is part of a broader shift from manual command execution to repeatable infrastructure practice.

Knowing the right commands is useful. Packaging those commands into clean, reusable operational tools is better.

That transition matters because strong infrastructure work is not only about knowing what to type. It is also about making common tasks repeatable, making verification quicker, and reducing friction during recovery and troubleshooting.

This folder represents that layer of the project.

---

## Current Status

The scripting layer is now in active use as part of the broader Ethereum node and monitoring buildout.

Current project progress:

- node setup completed
- monitoring stack completed
- dashboards completed
- documentation completed
- scripting layer implemented

---

## Next Direction

The next step for this directory is refinement rather than expansion.

Likely areas for future improvement include:

- cleaner pass/fail summaries
- optional colorized output
- more defensive error handling
- tighter metric parsing
- richer service dependency checks
- clearer exit codes for automation use later

For now, the focus remains on keeping the scripts practical, readable, and genuinely useful during real node operations.
