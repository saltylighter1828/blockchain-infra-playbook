# 🧑‍💻 Ethereum Node Operator Checklist
## Nethermind + Lighthouse + Prometheus + Grafana

A practical runbook for operating and monitoring an Ethereum node on Ubuntu under WSL2 using:

- **Nethermind** as the execution client
- **Lighthouse** as the consensus client
- **systemd** for service management
- **JWT-secured Engine API**
- **Node Exporter** for host metrics
- **Prometheus** for scraping
- **Grafana** for dashboards

---

## 📍 File Location

    nodes/operator-checklist.md

---

## 🗂️ Repo Structure

    blockchain-infra-playbook/
    ├── linux/
    ├── networking/
    ├── systemd/
    ├── monitoring/
    │   └── prometheus-grafana-setup.md
    ├── nodes/
    │   └── operator-checklist.md
    ├── assets/
    └── README.md

---

## 🎯 Operator Goals

1. Are the services alive?
2. Are execution and consensus talking?
3. Is sync progressing?
4. Is the machine healthy?

---

## 🧠 Current Stack Snapshot

### Core services
- nethermind
- lighthouse-beacon
- prometheus-node-exporter
- prometheus
- grafana-server

### Important paths
- Nethermind data: `/mnt/n/nethermind-data`
- Lighthouse data: `/var/lib/lighthouse`
- JWT secret: `/secrets/jwt.hex`
- Nethermind env: `/home/nethermind/.env`

### Important ports
- 8545 → JSON-RPC
- 8551 → Engine API
- 30303 → Nethermind P2P
- 9000 → Lighthouse P2P
- 6060 → Nethermind metrics
- 9100 → Node Exporter
- 9090 → Prometheus
- 3000 → Grafana

---

## 🟢 Daily Fast Check

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus-node-exporter --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager

    ss -tulpn | grep -E '8545|8551|30303|9000|6060|9100|9090|3000'

    sudo du -sh /mnt/n/nethermind-data
    sudo du -sh /var/lib/lighthouse

    free -h
    df -h /

    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

### Healthy looks like:
- services running
- ports listening
- data growing
- logs active
- memory stable

---

## 🟡 Sync Behaviour

### Good signs
- logs updating
- data growing
- peers present
- blocks processing

### Warning signs
- logs frozen
- no growth
- repeated errors
- memory exhaustion

---

## 👀 Live Monitoring

    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f
    journalctl -u prometheus -f
    journalctl -u grafana-server -f

---

## 🟠 Every Few Days

    df -h
    free -h
    nproc

    ps aux | grep nethermind
    ps aux | grep lighthouse

    curl -s http://127.0.0.1:9100/metrics | head
    curl -s http://127.0.0.1:6060/metrics | head
    curl http://127.0.0.1:9090/-/healthy
    curl http://127.0.0.1:3000/api/health

---

## 🔵 Weekly Deep Check

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

---

## 📊 Prometheus Expectations

Expected jobs:

- lighthouse
- nethermind
- node
- prometheus

Healthy state:
- all health = up
- all up = 1

---

## 🔐 JWT Check

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_ok || echo nethermind_fail
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_ok || echo lighthouse_fail

---

## 🧠 WSL2 Config

    [wsl2]
    memory=24GB
    processors=14
    swap=8GB

Restart WSL after change:

    wsl --shutdown

---

## 🔧 Common Incidents

### Service down
- check status
- check logs

### JWT issues
- check permissions
- check port 8551

### Prometheus issues
- check targets
- check exporters

### Grafana issues
- check port 3000
- check logs

---

## 🔁 Restart Rules

    sudo systemctl restart nethermind
    sudo systemctl restart lighthouse-beacon
    sudo systemctl restart prometheus
    sudo systemctl restart grafana-server

Always check logs before restart.

---

## ⏹️ Safe Shutdown (WSL)

    sudo systemctl stop nethermind lighthouse-beacon prometheus prometheus-node-exporter grafana-server
    systemctl status nethermind lighthouse-beacon prometheus grafana-server --no-pager
    exit

Then in Windows:

    wsl --shutdown

---

## ⚠️ Nethermind Stop Note

Stopping Nethermind may show:

    Active: failed

But logs show clean shutdown.

Exit code 130 = normal manual stop.

---

## ✅ Healthy Node Checklist

- services running
- ports listening
- logs active
- metrics scraping
- Grafana accessible
- memory stable
- disk safe

---

## ⚠️ Warning Signs

- failed services
- repeated restarts
- permission errors
- missing ports
- no sync progress
- high memory pressure

---

## 🔥 Daily Copy Block

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    ss -tulpn | grep -E '8545|8551|30303|9000|6060|9100|9090|3000'
    sudo du -sh /mnt/n/nethermind-data
    free -h
    df -h /
    journalctl -u nethermind -n 20 --no-pager

---

## 🚀 End Goal

- understand system behaviour
- monitor confidently
- debug methodically
- operate without guessing
