🧑‍💻 Ethereum Node Operator Checklist (Nethermind + Lighthouse + Prometheus + Grafana)

This document contains a practical operational checklist for running and monitoring an Ethereum node using:

Nethermind as the execution client
Lighthouse as the consensus client
systemd for service management
JWT-authenticated Engine API
Prometheus for metrics collection
Node Exporter for host metrics
Grafana for dashboards
📍 Where This File Lives

Create this file at:

nodes/operator-checklist.md
📁 Recommended Repo Structure
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
🧠 What This Checklist Covers

This checklist helps monitor:

execution + consensus client health
sync progress
ports and networking
logs and errors
storage growth
memory and swap pressure
Prometheus scrape health
Grafana availability
🟢 Daily Health Check (3 to 5 minutes)

Run this once per day:

systemctl status nethermind --no-pager
systemctl status lighthouse-beacon --no-pager
systemctl status prometheus-node-exporter --no-pager
systemctl status prometheus --no-pager
systemctl status grafana-server --no-pager

ss -tulpn | grep -E '8545|8551|30303|9000|9100|9090|3000'

sudo du -sh /home/nethermind/data
sudo du -sh /var/lib/lighthouse

free -h

journalctl -u nethermind -n 20 --no-pager
journalctl -u lighthouse-beacon -n 20 --no-pager
✅ Verify
all services show active (running)
ports are open:
8545 → JSON-RPC
8551 → Engine API
30303 → execution P2P
9000 → consensus P2P
9100 → Node Exporter
9090 → Prometheus
3000 → Grafana
Nethermind and Lighthouse data directories continue growing during sync
free -h shows healthy available memory
swap usage is low or stable
recent logs show normal activity and no critical errors
👀 Live Monitoring (Optional)

Use when debugging or watching sync or service behavior in real time:

journalctl -u nethermind -f
journalctl -u lighthouse-beacon -f
journalctl -u prometheus -f
journalctl -u grafana-server -f
🟡 Every Few Days (5 to 10 minutes)
df -h
free -h
nproc

ps aux | grep nethermind
ps aux | grep lighthouse
ps aux | grep prometheus
ps aux | grep grafana

curl http://127.0.0.1:9090/-/healthy
curl http://127.0.0.1:3000/api/health
✅ Verify
enough disk space remains
memory available is healthy
swap is not climbing badly
Prometheus returns Prometheus Server is Healthy.
Grafana health API returns a JSON response with database ok
all core processes are still running
🔵 Weekly Deep Check (10 to 15 minutes)
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

curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
curl -s "http://127.0.0.1:9090/api/v1/query?query=up"
✅ What to Look For
no repeated crashes
no restart loops
no permission errors
no JWT / Engine API authentication failures
logs show continued syncing and block processing
Prometheus targets show both prometheus and node as up
up query returns 1 for both scrape targets
disk growth is consistent
no worrying increase in swap use
📊 Monitoring Stack Health Check

Use this when you want to confirm the observability stack itself is alive.

Node Exporter
systemctl status prometheus-node-exporter --no-pager
ss -tulpn | grep 9100
curl http://127.0.0.1:9100/metrics | head
Prometheus
systemctl status prometheus --no-pager
ss -tulpn | grep 9090
curl http://127.0.0.1:9090/-/healthy
curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
curl -s "http://127.0.0.1:9090/api/v1/query?query=up"
Grafana
systemctl status grafana-server --no-pager
ss -tulpn | grep 3000
curl http://127.0.0.1:3000/api/health
✅ Verify
Node Exporter serves metrics on 9100
Prometheus health endpoint responds successfully
Prometheus shows node and prometheus targets as healthy
Grafana health endpoint returns JSON with database ok
🧠 WSL2 Resource Check

Because this node runs under WSL2, memory allocation matters.

Run:

free -h
df -h
hostname -I
✅ Verify
WSL has enough RAM available
swap is not being hammered
root disk still has plenty of free space
you know the current WSL IP if browser access to local services is needed
Notes
High swap use usually means WSL memory is too tight or the workload is heavy
A little swap is okay
Bad signs are:
high swap
low available RAM
sluggish system
If needed, tune WSL with a Windows-side .wslconfig

Example:

[wsl2]
memory=24GB
processors=8
swap=8GB
🔧 Debugging Commands (When Something Breaks)
Client Logs
journalctl -u nethermind -f
journalctl -u lighthouse-beacon -f
Monitoring Logs
journalctl -u prometheus -f
journalctl -u grafana-server -f
Port Check
ss -tulpn | grep -E '8545|8551|30303|9000|9100|9090|3000'
JWT Readability Check
sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_ok || echo nethermind_fail
sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_ok || echo lighthouse_fail
Prometheus API Check
curl http://127.0.0.1:9090/-/healthy
curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
curl -s "http://127.0.0.1:9090/api/v1/query?query=up"
Grafana API Check
curl http://127.0.0.1:3000/api/health
Get WSL IP
hostname -I
🔁 Restart Services (if Required)
sudo systemctl restart nethermind
sudo systemctl restart lighthouse-beacon
sudo systemctl restart prometheus-node-exporter
sudo systemctl restart prometheus
sudo systemctl restart grafana-server
🧠 Healthy Node Indicators
Nethermind and Lighthouse both consistently show active (running)
Lighthouse logs show peers, synced state, and new blocks
Nethermind logs show block processing and forkchoice updates
execution and consensus clients continue talking to each other
ports remain open and stable
disk usage grows during sync
Prometheus is scraping successfully
Grafana is reachable and healthy
available memory is healthy and swap remains under control
⚠️ Warning Signs

Investigate if you see:

failed service status
repeated restarts
permission denied
JWT / Engine API authentication failures
ports not listening
disk usage stops growing unexpectedly
Prometheus target health not up
Grafana health endpoint failing
excessive memory use combined with rising swap
system feels slow or logs stall
📌 Notes
This node runs on Ubuntu under WSL2
systemd manages startup and restart behavior
JWT secret is required for execution ↔ consensus communication
WSL memory tuning can significantly improve node stability
Windows browser access to Prometheus or Grafana may require the WSL IP instead of localhost
Grafana local default login is:
username: admin
password: admin
🚀 Goal

Build operational confidence managing blockchain infrastructure by learning how to:

monitor node health
understand logs
watch memory and disk behavior
validate observability tooling
debug failures quickly
develop production-style operating habits
🔥 Quick Copy-Paste Daily Block
systemctl status nethermind --no-pager
systemctl status lighthouse-beacon --no-pager
systemctl status prometheus-node-exporter --no-pager
systemctl status prometheus --no-pager
systemctl status grafana-server --no-pager
ss -tulpn | grep -E '8545|8551|30303|9000|9100|9090|3000'
sudo du -sh /home/nethermind/data
sudo du -sh /var/lib/lighthouse
free -h
journalctl -u nethermind -n 20 --no-pager
journalctl -u lighthouse-beacon -n 20 --no-pager
🔥 Quick Copy-Paste Weekly Block
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
curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
curl -s "http://127.0.0.1:9090/api/v1/query?query=up"
curl http://127.0.0.1:3000/api/health
✅ Summary

This file belongs at:

nodes/operator-checklist.md

And it documents the real day-to-day workflow for operating and monitoring a full Ethereum node with:

Nethermind
Lighthouse
systemd
JWT-secured Engine API
Node Exporter
Prometheus
Grafana
