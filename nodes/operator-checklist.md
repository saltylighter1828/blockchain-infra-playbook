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

Save this file at:

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

As an operator, I want to know four things fast:

1. **Are the services alive?**
2. **Are execution and consensus talking to each other?**
3. **Is sync progressing normally?**
4. **Is the machine staying healthy?**

If I can answer those four quickly, I’m operating instead of guessing.

---

## 🧠 Current Stack Snapshot

### Core services
- `nethermind`
- `lighthouse-beacon`
- `prometheus-node-exporter`
- `prometheus`
- `grafana-server`

### Important paths
- Nethermind data: `/home/nethermind/data`
- Lighthouse data: `/var/lib/lighthouse`
- JWT secret: `/secrets/jwt.hex`
- Nethermind env: `/home/nethermind/.env`

### Important ports
- `8545` → Nethermind JSON-RPC
- `8551` → Engine API
- `30303` → Nethermind P2P
- `9000` → Lighthouse P2P
- `9100` → Node Exporter
- `9090` → Prometheus
- `3000` → Grafana

---

## 🟢 Daily Fast Check (3 to 5 minutes)

Run once per day:

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus-node-exporter --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager

    ss -tulpn | grep -E '8545|8551|30303|9000|9100|9090|3000'

    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse

    free -h
    df -h /

    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

### ✅ What “healthy enough” looks like
- all services show `active (running)`
- expected ports are listening
- data directories are still growing during sync
- available memory is reasonable
- swap is low or at least not climbing badly
- logs show ordinary activity, not repeated failures
- Nethermind logs show block activity / engine activity
- Lighthouse logs show peers / sync / head movement

---

## 🟡 If the Node Is Still Syncing

Use these signs to avoid false panic:

### Good syncing signs
- data directory size keeps increasing
- logs continue updating
- Lighthouse shows peers and head movement
- Nethermind keeps processing blocks / syncing state
- system is busy but not choking

### Worrying signs
- logs stop changing for a long time
- no data growth
- repeated auth / JWT / engine errors
- repeated crashes or restart loops
- machine becomes memory-starved and swap starts ballooning

### Extra sync watch
    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f

---

## 👀 Live Monitoring

Use during troubleshooting or when you simply want to watch the node breathe:

    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f
    journalctl -u prometheus -f
    journalctl -u grafana-server -f

---

## 🟠 Every Few Days (5 to 10 minutes)

    df -h
    free -h
    nproc

    ps aux | grep nethermind
    ps aux | grep lighthouse
    ps aux | grep prometheus
    ps aux | grep grafana

    curl http://127.0.0.1:9100/metrics | head
    curl http://127.0.0.1:9090/-/healthy
    curl http://127.0.0.1:3000/api/health

### ✅ Verify
- root disk still has room
- memory looks stable
- swap is not quietly creeping upward forever
- Node Exporter responds
- Prometheus returns healthy
- Grafana returns JSON health with database `ok`
- processes are present and not zombie-like ghosts in a trench coat

---

## 🔵 Weekly Deep Check (10 to 15 minutes)

### Full service state
    systemctl status nethermind --no-pager -l
    systemctl status lighthouse-beacon --no-pager -l
    systemctl status prometheus-node-exporter --no-pager -l
    systemctl status prometheus --no-pager -l
    systemctl status grafana-server --no-pager -l

### Longer logs
    journalctl -u nethermind -n 100 --no-pager
    journalctl -u lighthouse-beacon -n 100 --no-pager
    journalctl -u prometheus -n 50 --no-pager
    journalctl -u grafana-server -n 50 --no-pager

### Storage + memory
    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse
    df -h
    free -h

### Prometheus target checks
    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

### Config snapshot checks
    systemctl cat nethermind
    systemctl cat lighthouse-beacon
    sudo cat /home/nethermind/.env
    ls -l /secrets/jwt.hex

### ✅ What to look for
- no repeated crash loops
- no permission errors
- no JWT / Engine API auth failures
- no broken service file edits
- Prometheus targets show `up`
- `up` query returns `1` for expected jobs
- pruning settings still look correct
- JWT file still exists and permissions still make sense
- disk usage is growing in a way that matches expected node behaviour

---

## 📊 Dashboard Review Habit

Use Grafana as a cockpit, not wall art.

### Lighthouse dashboard
Check these first:
- sync status
- head slot
- finalized value
- peer count
- health panel
- whether values are moving forward over time

### Nethermind dashboard
As you build it out, check:
- process up / health
- block progression
- peer count
- RPC responsiveness
- sync state
- system resource pressure

### Operator rule
A single frozen-looking panel is not enough to panic.
Correlate:
- dashboard movement
- logs
- service status
- disk growth

Three clues beat one spooky graph.

---

## 📦 Pruning / Storage Check

Because you configured Nethermind pruning, review this weekly:

    sudo cat /home/nethermind/.env | grep PRUNING

### Confirm expected values are present
Examples:
- `NETHERMIND_PRUNINGCONFIG_MODE=Hybrid`
- `NETHERMIND_PRUNINGCONFIG_FULLPRUNINGTRIGGER=VolumeFreeSpace`
- `NETHERMIND_PRUNINGCONFIG_FULLPRUNINGTHRESHOLDMB=400000`

### Watch for
- root disk getting tight
- data growth much faster than expected
- pruning config accidentally removed during edits

---

## 🧠 WSL2 Host Check

This node runs under WSL2, so host resources matter.

Run:

    free -h
    df -h
    hostname -I

### ✅ Verify
- RAM available is still healthy
- swap is not being hammered
- root disk still has comfortable free space
- you know the current WSL IP if browser access needs it

### Notes
- a little swap is okay
- high swap + low available RAM + sluggishness = danger trio
- if needed, tune Windows-side `.wslconfig`

Example:

    [wsl2]
    memory=24GB
    processors=8
    swap=8GB

---

## 🔧 Incident Playbooks

### 1. Service is down
    systemctl status nethermind --no-pager -l
    systemctl status lighthouse-beacon --no-pager -l
    journalctl -u nethermind -n 50 --no-pager
    journalctl -u lighthouse-beacon -n 50 --no-pager

Check:
- bad config edit
- missing file
- permission issue
- crash loop
- resource exhaustion

---

### 2. JWT / Engine API issue
    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_ok || echo nethermind_fail
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_ok || echo lighthouse_fail
    ls -l /secrets/jwt.hex
    ss -tulpn | grep 8551
    journalctl -u nethermind -n 50 --no-pager
    journalctl -u lighthouse-beacon -n 50 --no-pager

Check:
- JWT file exists
- both service users can read it
- Engine API port `8551` is listening
- Lighthouse is pointing to the right execution endpoint
- auth failures are not appearing in logs

---

### 3. Prometheus says target is down
    curl http://127.0.0.1:9090/-/healthy
    curl -s http://127.0.0.1:9090/api/v1/targets
    systemctl status prometheus --no-pager -l
    systemctl status prometheus-node-exporter --no-pager -l

Check:
- Prometheus service alive
- Node Exporter alive
- scrape targets are still correct
- no config typo in Prometheus

---

### 4. Grafana not loading
    systemctl status grafana-server --no-pager -l
    journalctl -u grafana-server -n 50 --no-pager
    ss -tulpn | grep 3000
    curl http://127.0.0.1:3000/api/health

Check:
- Grafana service alive
- port `3000` listening
- health endpoint returns JSON
- if using browser from Windows, test WSL IP too

---

### 5. Sync looks stalled
    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f
    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse
    free -h

Check:
- are logs still moving?
- are data dirs still growing?
- is memory exhausted?
- are services still running even if progress is slow?
- is it actually stalled, or just in a slower sync phase?

---

## 🔁 Restart Rules

Only restart after checking logs first.

### Safe restart commands
    sudo systemctl restart nethermind
    sudo systemctl restart lighthouse-beacon
    sudo systemctl restart prometheus-node-exporter
    sudo systemctl restart prometheus
    sudo systemctl restart grafana-server

### Good operator habit
Before restart:
1. check status
2. check recent logs
3. identify likely reason
4. restart once
5. re-check logs immediately

Do not turn “restart it” into a religion.

---

## ⏹️ Planned Shutdown / Maintenance

If intentionally shutting the machine down:

### Optional clean stop
    sudo systemctl stop lighthouse-beacon
    sudo systemctl stop nethermind
    sudo systemctl stop prometheus
    sudo systemctl stop grafana-server
    sudo systemctl stop prometheus-node-exporter

Then shut down WSL / PC normally.

### Startup check after boot
    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager
    ss -tulpn | grep -E '8551|9000|9090|3000'

---

## 🧾 Evidence / Documentation Habit

When something meaningful changes, capture evidence for your repo:

### Good screenshot targets
- `systemctl status nethermind`
- `systemctl status lighthouse-beacon`
- Lighthouse syncing logs
- Nethermind block / forkchoice logs
- `up` query returning `1`
- Prometheus targets healthy
- Grafana dashboard showing real data
- pruning config in `.env`
- JWT readability checks passing

### Good markdown notes
Record:
- what changed
- why you changed it
- commands used
- errors seen
- final working state
- lessons learned

That turns random debugging into portfolio gold.

---

## ✅ Healthy Node Indicators

A healthy node usually looks like this:

- Nethermind is running
- Lighthouse is running
- execution and consensus are communicating
- expected ports are listening
- logs are alive and not screaming
- data grows while syncing
- Prometheus scrapes successfully
- Grafana is reachable
- memory is okay
- swap is controlled
- disk is not sneaking toward disaster

---

## ⚠️ Warning Signs

Investigate quickly if you see:

- `failed` service status
- repeated restarts
- `permission denied`
- JWT / Engine API auth failures
- missing port listeners
- no data growth during expected sync activity
- Prometheus target health not `up`
- Grafana health endpoint failing
- memory pressure with rising swap
- machine feels slow and logs go quiet

---

## 🔥 Daily Copy-Paste Block

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus-node-exporter --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager
    ss -tulpn | grep -E '8545|8551|30303|9000|9100|9090|3000'
    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse
    free -h
    df -h /
    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

---

## 🔥 Weekly Copy-Paste Block

    systemctl status nethermind --no-pager -l
    systemctl status lighthouse-beacon --no-pager -l
    systemctl status prometheus-node-exporter --no-pager -l
    systemctl status prometheus --no-pager -l
    systemctl status grafana-server --no-pager -l
    journalctl -u nethermind -n 100 --no-pager
    journalctl -u lighthouse-beacon -n 100 --no-pager
    journalctl -u prometheus -n 50 --no-pager
    journalctl -u grafana-server -n 50 --no-pager
    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse
    df -h
    free -h
    sudo cat /home/nethermind/.env | grep PRUNING
    ls -l /secrets/jwt.hex
    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"
    curl http://127.0.0.1:3000/api/health

---

## 🚀 End Goal

Build real operator confidence by learning how to:

- monitor node health
- read logs without fear
- validate execution ↔ consensus communication
- watch disk, memory, and swap like an adult
- use dashboards as confirmation, not decoration
- debug methodically
- document work like a professional infra engineer
