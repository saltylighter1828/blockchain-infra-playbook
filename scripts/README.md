# 🧑‍💻 Ethereum Node Operator Toolkit
## Nethermind + Lighthouse + Prometheus + Grafana

A practical GitHub README-style runbook and script pack for operating and monitoring an Ethereum node on Ubuntu under WSL2 using:

- **Nethermind** as the execution client
- **Lighthouse** as the consensus client
- **systemd** for service management
- **JWT-secured Engine API**
- **Prometheus** for metrics scraping
- **Node Exporter** for host metrics
- **Grafana** for dashboards and observability

---

## 🎯 Goal

This toolkit is designed to help build real operator habits by making it easier to:

- check whether services are alive
- confirm execution and consensus are communicating
- inspect logs quickly
- watch disk and memory health
- validate Prometheus and Grafana health
- restart services cleanly and verify recovery
- document a real node operator workflow in a portfolio

---

## 📁 Recommended Repo Structure

    blockchain-infra-playbook/
    ├── linux/
    ├── networking/
    ├── systemd/
    ├── monitoring/
    │   └── prometheus-grafana-setup.md
    ├── nodes/
    │   └── operator-checklist.md
    ├── scripts/
    │   ├── health-check.sh
    │   ├── service-status.sh
    │   ├── disk-memory-check.sh
    │   ├── quick-log-summary.sh
    │   ├── restart-verify.sh
    │   └── daily-operator-check.sh
    ├── assets/
    └── README.md

---

## 🧠 What This Covers

This operator toolkit helps monitor:

- execution client health
- consensus client health
- monitoring stack health
- sync activity
- recent logs
- ports and networking
- memory and swap pressure
- disk growth and available space
- Prometheus scrape health
- Grafana API health

---

## ⚙️ Core Services

### Main node services
- `nethermind`
- `lighthouse-beacon`

### Monitoring services
- `prometheus-node-exporter`
- `prometheus`
- `grafana-server`

---

## 📍 Important Paths

- Nethermind data directory: `/home/nethermind/data`
- Lighthouse data directory: `/var/lib/lighthouse`
- JWT secret: `/secrets/jwt.hex`
- Nethermind environment file: `/home/nethermind/.env`

---

## 🌐 Important Ports

- `8545` → Nethermind JSON-RPC
- `8551` → Engine API
- `30303` → Nethermind P2P
- `9000` → Lighthouse P2P
- `9100` → Node Exporter
- `9090` → Prometheus
- `3000` → Grafana

---

## 🚀 Operator Workflow

### Daily
- check service health
- confirm key ports are listening
- review recent Nethermind and Lighthouse logs
- confirm memory and disk are healthy
- confirm Prometheus and Grafana are reachable

### Every few days
- inspect data directory growth
- review scrape health
- confirm system still has comfortable resources

### Weekly
- inspect longer logs
- review pruning config
- check JWT file state
- validate dashboards and metrics are still meaningful

---

## 🛠️ Setup

Create the `scripts/` directory:

    mkdir -p scripts

Make all scripts executable:

    chmod +x scripts/service-status.sh
    chmod +x scripts/disk-memory-check.sh
    chmod +x scripts/quick-log-summary.sh
    chmod +x scripts/health-check.sh
    chmod +x scripts/restart-verify.sh
    chmod +x scripts/daily-operator-check.sh

Run from repo root or from inside `scripts/`.

---

## ▶️ Script Usage

### Common commands

    ./scripts/service-status.sh
    ./scripts/disk-memory-check.sh
    ./scripts/quick-log-summary.sh
    ./scripts/quick-log-summary.sh 50
    ./scripts/health-check.sh
    ./scripts/restart-verify.sh
    ./scripts/daily-operator-check.sh

### Most useful daily workflow

    ./scripts/daily-operator-check.sh

### Faster debugging workflow

    ./scripts/quick-log-summary.sh 50

### Controlled restart workflow

    ./scripts/restart-verify.sh

---

# 📜 Script Pack

## `scripts/service-status.sh`

Purpose:
- show current `systemd` status for all core services

Contents:

    #!/usr/bin/env bash
    set -u

    SERVICES=(
      nethermind
      lighthouse-beacon
      prometheus-node-exporter
      prometheus
      grafana-server
    )

    echo "=== SERVICE STATUS ==="
    for service in "${SERVICES[@]}"; do
      echo
      echo "--- $service ---"
      systemctl status "$service" --no-pager
    done

---

## `scripts/disk-memory-check.sh`

Purpose:
- inspect root disk, memory, and node data directory sizes

Contents:

    #!/usr/bin/env bash
    set -u

    NETHERMIND_DATA="/home/nethermind/data"
    LIGHTHOUSE_DATA="/var/lib/lighthouse"

    echo "=== DISK USAGE ==="
    df -h
    echo
    echo "=== ROOT DISK ==="
    df -h /

    echo
    echo "=== DATA DIRECTORY SIZES ==="
    sudo du -sh "$NETHERMIND_DATA" 2>/dev/null || echo "Could not read $NETHERMIND_DATA"
    sudo du -sh "$LIGHTHOUSE_DATA" 2>/dev/null || echo "Could not read $LIGHTHOUSE_DATA"

    echo
    echo "=== MEMORY ==="
    free -h

---

## `scripts/quick-log-summary.sh`

Purpose:
- show a short recent log summary for the main services
- accepts an optional line count argument
- default is `20`

Contents:

    #!/usr/bin/env bash
    set -u

    LINES="${1:-20}"

    SERVICES=(
      nethermind
      lighthouse-beacon
      prometheus
      grafana-server
    )

    echo "=== QUICK LOG SUMMARY (last $LINES lines) ==="

    for service in "${SERVICES[@]}"; do
      echo
      echo "--- $service ---"
      journalctl -u "$service" -n "$LINES" --no-pager
    done

Usage examples:

    ./scripts/quick-log-summary.sh
    ./scripts/quick-log-summary.sh 50

---

## `scripts/health-check.sh`

Purpose:
- run a consolidated node health check covering:
  - service states
  - expected ports
  - data directory sizes
  - memory
  - root disk
  - Prometheus health
  - Grafana health
  - recent Nethermind and Lighthouse logs

Contents:

    #!/usr/bin/env bash
    set -u

    PORT_PATTERN='8545|8551|30303|9000|9100|9090|3000'
    NETHERMIND_DATA="/home/nethermind/data"
    LIGHTHOUSE_DATA="/var/lib/lighthouse"

    SERVICES=(
      nethermind
      lighthouse-beacon
      prometheus-node-exporter
      prometheus
      grafana-server
    )

    echo "========================================"
    echo "ETHEREUM NODE HEALTH CHECK"
    echo "========================================"

    echo
    echo "=== 1. SERVICE STATES ==="
    for service in "${SERVICES[@]}"; do
      if systemctl is-active --quiet "$service"; then
        echo "[OK]    $service is active"
      else
        echo "[FAIL]  $service is NOT active"
      fi
    done

    echo
    echo "=== 2. PORT CHECK ==="
    ss -tulpn | grep -E "$PORT_PATTERN" || echo "[WARN] Expected ports not found"

    echo
    echo "=== 3. DATA DIRECTORY SIZES ==="
    sudo du -sh "$NETHERMIND_DATA" 2>/dev/null || echo "[WARN] Could not read $NETHERMIND_DATA"
    sudo du -sh "$LIGHTHOUSE_DATA" 2>/dev/null || echo "[WARN] Could not read $LIGHTHOUSE_DATA"

    echo
    echo "=== 4. MEMORY ==="
    free -h

    echo
    echo "=== 5. ROOT DISK ==="
    df -h /

    echo
    echo "=== 6. PROMETHEUS HEALTH ==="
    curl -fsS http://127.0.0.1:9090/-/healthy && echo || echo "[FAIL] Prometheus health check failed"

    echo
    echo "=== 7. GRAFANA HEALTH ==="
    curl -fsS http://127.0.0.1:3000/api/health && echo || echo "[FAIL] Grafana health check failed"

    echo
    echo "=== 8. PROMETHEUS UP QUERY ==="
    curl -fsS "http://127.0.0.1:9090/api/v1/query?query=up" || echo "[FAIL] Could not query Prometheus"

    echo
    echo "=== 9. RECENT NETHERMIND LOGS ==="
    journalctl -u nethermind -n 10 --no-pager

    echo
    echo "=== 10. RECENT LIGHTHOUSE LOGS ==="
    journalctl -u lighthouse-beacon -n 10 --no-pager

---

## `scripts/restart-verify.sh`

Purpose:
- restart services one by one
- immediately verify whether each service returned to `active`
- print recent logs if a restart fails

Contents:

    #!/usr/bin/env bash
    set -u

    SERVICES=(
      nethermind
      lighthouse-beacon
      prometheus-node-exporter
      prometheus
      grafana-server
    )

    echo "=== RESTART AND VERIFY ==="

    for service in "${SERVICES[@]}"; do
      echo
      echo "--- Restarting $service ---"
      sudo systemctl restart "$service"

      if systemctl is-active --quiet "$service"; then
        echo "[OK]    $service restarted successfully"
      else
        echo "[FAIL]  $service failed to restart"
        echo "Recent logs for $service:"
        journalctl -u "$service" -n 20 --no-pager
      fi
    done

---

## `scripts/daily-operator-check.sh`

Purpose:
- run the main daily operator routine in one command
- combines service status, disk/memory checks, quick logs, and health checks

Contents:

    #!/usr/bin/env bash
    set -u

    SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

    echo "========================================"
    echo "DAILY OPERATOR CHECK"
    echo "========================================"
    echo

    bash "$SCRIPT_DIR/service-status.sh"
    echo
    bash "$SCRIPT_DIR/disk-memory-check.sh"
    echo
    bash "$SCRIPT_DIR/quick-log-summary.sh" 20
    echo
    bash "$SCRIPT_DIR/health-check.sh"

---

# ✅ Healthy Node Indicators

A healthy stack usually looks like this:

- Nethermind and Lighthouse both show `active (running)`
- expected ports are listening
- Lighthouse logs show peers, sync activity, or head movement
- Nethermind logs show block processing, sync activity, or engine communication
- Prometheus health endpoint responds successfully
- Grafana health endpoint returns JSON with database `ok`
- available memory is healthy
- swap is low or stable
- data directories continue growing during sync
- dashboards reflect movement rather than frozen values

---

# ⚠️ Warning Signs

Investigate when you see:

- `failed` service states
- repeated restart loops
- `permission denied`
- missing or unreadable JWT secret
- Engine API authentication failures
- expected ports not listening
- Prometheus targets not `up`
- Grafana API health failing
- memory pressure with rising swap
- logs stalling unexpectedly
- data growth stopping during expected sync phases

---

# 🔍 Good Debugging Commands

## Service and status

    systemctl status nethermind --no-pager -l
    systemctl status lighthouse-beacon --no-pager -l
    systemctl status prometheus --no-pager -l
    systemctl status grafana-server --no-pager -l

## Recent logs

    journalctl -u nethermind -n 50 --no-pager
    journalctl -u lighthouse-beacon -n 50 --no-pager
    journalctl -u prometheus -n 50 --no-pager
    journalctl -u grafana-server -n 50 --no-pager

## Live logs

    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f
    journalctl -u prometheus -f
    journalctl -u grafana-server -f

## Ports

    ss -tulpn | grep -E '8545|8551|30303|9000|9100|9090|3000'

## Prometheus API

    curl http://127.0.0.1:9090/-/healthy
    curl -s http://127.0.0.1:9090/api/v1/targets
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

## Grafana API

    curl http://127.0.0.1:3000/api/health

## JWT readability

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_ok || echo nethermind_fail
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_ok || echo lighthouse_fail
    ls -l /secrets/jwt.hex

## System resources

    df -h
    df -h /
    free -h
    hostname -I

---

# 🧪 Suggested Operator Routine

## Daily
Run:

    ./scripts/daily-operator-check.sh

Look for:
- service health
- listening ports
- recent log activity
- memory and disk sanity
- Prometheus/Grafana responding

## When something feels wrong
Run:

    ./scripts/quick-log-summary.sh 50
    ./scripts/health-check.sh

## After config edits or maintenance
Run:

    ./scripts/restart-verify.sh

---

# 📊 Dashboard Review Habit

Once Grafana is working, use it like an operator:

## Host dashboard
Check:
- CPU
- RAM
- disk
- network activity

## Lighthouse dashboard
Check:
- head slot
- finalized value
- peer count
- sync movement
- health panel

## Nethermind dashboard
Check:
- process health
- sync or block progression
- peer count
- node responsiveness
- resource pressure

Operator rule:
- never trust one panel by itself
- correlate dashboards with logs, service status, and disk growth

---

# 🧠 WSL2 Notes

Because this node runs under WSL2:

- RAM allocation matters
- swap pressure matters
- Windows-side resource limits can affect node stability
- browser access to services may sometimes require the WSL IP instead of `localhost`

Useful commands:

    free -h
    df -h
    hostname -I

Example `.wslconfig`:

    [wsl2]
    memory=24GB
    processors=8
    swap=8GB

---

# 🧾 Portfolio Value

This toolkit demonstrates practical experience with:

- Linux fundamentals
- `systemd`
- log inspection
- service operations
- node monitoring
- Ethereum execution and consensus architecture
- JWT-authenticated client communication
- Prometheus and Grafana health validation
- operator-style debugging and documentation

---

# 🚀 Summary

This script pack and runbook help turn a working Ethereum node into a repeatable operator workflow.

It gives a practical way to:

- monitor the full stack
- debug quickly
- build daily operational habits
- document a real infra portfolio project
- show evidence of understanding rather than just screenshots

If added to a GitHub repository, this can serve as both:
- a personal operating guide
- a portfolio artifact for blockchain infra or node operator roles
