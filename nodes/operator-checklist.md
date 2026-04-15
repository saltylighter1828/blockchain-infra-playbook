# 🧑‍💻 Ethereum Node Operator Checklist
## Nethermind + Lighthouse + Prometheus + Grafana

A practical operations runbook for managing and monitoring an Ethereum node stack on Ubuntu under WSL2.

This checklist is designed for repeatable day-to-day operations across the local execution, consensus, and observability stack, with an emphasis on verification, controlled recovery, and clear system visibility.

---

## 📍 File Location

    nodes/operator-checklist.md

---

## 🗂️ Repository Context

    blockchain-infra-playbook/
    ├── linux/
    ├── networking/
    ├── systemd/
    ├── monitoring/
    │   └── prometheus-grafana-setup.md
    ├── nodes/
    │   └── operator-checklist.md
    ├── scripts/
    ├── assets/
    └── README.md

---

## 🎯 Operating Priorities

Every check in this runbook is built around four core questions:

1. Are the services healthy?
2. Are execution and consensus communicating correctly?
3. Is sync progressing normally?
4. Is the host machine stable enough to support the stack?

---

## 🧠 Current Stack Snapshot

### Core Services
- `nethermind`
- `lighthouse-beacon`
- `prometheus-node-exporter`
- `prometheus`
- `grafana-server`

### Important Paths
- Nethermind data: `/mnt/n/nethermind-data`
- Lighthouse data: `/var/lib/lighthouse`
- JWT secret: `/secrets/jwt.hex`
- Nethermind environment file: `/home/nethermind/.env`

### Important Ports
- `8545` → JSON-RPC
- `8551` → Engine API
- `30303` → Nethermind P2P
- `9000` → Lighthouse P2P
- `5054` → Lighthouse metrics
- `6060` → Nethermind metrics
- `9100` → Node Exporter
- `9090` → Prometheus
- `3000` → Grafana

---

## 🟢 Daily Fast Check

Use this when you want a quick operational snapshot of the full stack.

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus-node-exporter --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager

    ss -tulpn | grep -E '8545|8551|30303|9000|5054|6060|9100|9090|3000'

    sudo du -sh /mnt/n/nethermind-data
    sudo du -sh /var/lib/lighthouse

    free -h
    df -h /

    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

### Healthy Looks Like
- services are `active (running)`
- expected ports are listening
- Nethermind and Lighthouse data directories continue to grow appropriately
- recent logs show normal activity
- memory and disk usage remain stable

---

## 🟡 Sync Behaviour

### Good Signs
- logs continue to update
- peers are present
- block processing continues
- sync distance decreases over time
- head slot and execution block height continue moving forward

### Warning Signs
- logs stop changing for long periods
- no visible data growth
- repeated sync resets or recurring errors
- persistent block lag
- memory pressure or host instability

---

## 👀 Live Monitoring

Use these commands when actively observing service behaviour during sync, restart, or troubleshooting.

    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f
    journalctl -u prometheus -f
    journalctl -u grafana-server -f

---

## 🟠 Every Few Days

Run these checks to review general system health and endpoint behaviour.

    df -h
    free -h
    nproc

    ps aux | grep nethermind
    ps aux | grep lighthouse

    curl -s http://127.0.0.1:9100/metrics | head
    curl -s http://127.0.0.1:6060/metrics | head
    curl -s http://127.0.0.1:5054/metrics | head
    curl http://127.0.0.1:9090/-/healthy
    curl http://127.0.0.1:3000/api/health

### Review Focus
- host resource usage
- process presence
- metrics endpoint responsiveness
- Prometheus health
- Grafana API health

---

## 🔵 Weekly Deep Check

Use this section for a more complete inspection of service state, logs, storage growth, and monitoring health.

    systemctl status nethermind --no-pager -l
    systemctl status lighthouse-beacon --no-pager -l

    journalctl -u nethermind -n 100 --no-pager
    journalctl -u lighthouse-beacon -n 100 --no-pager

    sudo du -sh /mnt/n/nethermind-data
    sudo du -sh /var/lib/lighthouse

    df -h
    free -h

    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

### Weekly Review Focus
- longer service status output
- recent warning patterns in logs
- storage growth trends
- memory and disk safety margins
- Prometheus scrape health across all configured jobs

---

## 📊 Prometheus Expectations

### Expected Jobs
- `lighthouse`
- `nethermind`
- `node`
- `prometheus`

### Healthy State
- all targets report `"health":"up"`
- all expected `up` series return value `1`

### Useful Verification Commands

    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

---

## 🔐 JWT Check

Use this when validating execution-consensus authentication.

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_ok || echo nethermind_fail
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_ok || echo lighthouse_fail

### Healthy State
- both checks return `*_ok`
- Lighthouse can read the same JWT secret used by Nethermind
- Engine API communication on `8551` succeeds

---

## 🧠 WSL2 Configuration

Current runtime profile:

    [wsl2]
    memory=24GB
    processors=14
    swap=8GB

After updating `.wslconfig`, restart WSL from Windows:

    wsl --shutdown

---

## 🔧 Common Incident Patterns

### Service Down
Check:
- `systemctl status`
- recent `journalctl` output
- expected ports
- any recent config changes

### JWT / Engine API Issues
Check:
- JWT file permissions
- Nethermind Engine API on `8551`
- Lighthouse execution endpoint settings
- logs for authentication or forkchoice issues

### Prometheus Issues
Check:
- Prometheus service health
- `/targets` output
- exporter endpoints
- missing or failed scrape jobs

### Grafana Issues
Check:
- port `3000`
- `grafana-server` status
- Grafana logs
- Prometheus datasource connectivity

### Sync Issues
Check:
- peer count
- execution block progression
- Lighthouse sync distance
- Nethermind block lag
- recent error loops in logs

---

## 🔁 Restart Rules

Restart deliberately, not reflexively.

### Controlled Restart Sequence

    sudo systemctl restart nethermind
    sudo systemctl restart lighthouse-beacon
    sudo systemctl restart prometheus
    sudo systemctl restart grafana-server

### Restart Principles
- inspect logs before restarting
- restart the minimum number of services required
- verify service state after restart
- confirm expected ports and metrics endpoints return
- treat recovery as incomplete until health is re-verified

---

## ⏹️ Safe Shutdown (WSL)

Use this when intentionally pausing the stack.

    sudo systemctl stop nethermind lighthouse-beacon prometheus prometheus-node-exporter grafana-server
    systemctl status nethermind lighthouse-beacon prometheus grafana-server --no-pager
    exit

Then from Windows:

    wsl --shutdown

### Preferred Principle
Stop services cleanly before shutting down the WSL VM.

---

## ⚠️ Nethermind Stop Note

Stopping Nethermind may sometimes show:

    Active: failed

This can still correspond to an intentional manual stop rather than an unhealthy crash.

### Important Context
- logs may still show a clean shutdown sequence
- exit code `130` can represent a normal manual interruption
- verify the stop behaviour through logs and follow-up service state rather than relying on a single status line in isolation

---

## ✅ Healthy Node Checklist

A healthy stack usually shows the following:

- services are running
- expected ports are listening
- logs are active and current
- Prometheus is scraping targets successfully
- Grafana is reachable
- execution and consensus remain in sync
- memory usage is stable
- disk usage remains within safe limits

---

## ⚠️ Warning Signs

Investigate promptly if you observe any of the following:

- failed services
- repeated restarts
- missing ports
- authentication or permission errors
- stalled logs
- no sync progress
- persistent block lag
- high memory pressure
- disk nearing unsafe capacity
- scrape targets reporting down

---

## 🔥 Daily Copy Block

Use this when you want a compact daily check sequence.

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    ss -tulpn | grep -E '8545|8551|30303|9000|5054|6060|9100|9090|3000'
    sudo du -sh /mnt/n/nethermind-data
    free -h
    df -h /
    journalctl -u nethermind -n 20 --no-pager

---

## 🧭 Operating Principle

This checklist is designed to support a simple operating model:

- observe before acting
- verify before assuming
- recover in a controlled order
- validate with logs, ports, and metrics
- document what changed and what was learned

---

## 🚀 End Goal

The purpose of this runbook is to build the habits required to operate infrastructure confidently:

- understand system behaviour
- monitor services and metrics with confidence
- debug methodically across execution, consensus, and host layers
- recover services in a controlled way
- operate without guessing
