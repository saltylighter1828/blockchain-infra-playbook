# 🧑‍💻 Ethereum Node Operator Checklist (Nethermind + Lighthouse)

This document contains a complete guide and operational checklist for running and monitoring an Ethereum node using:

- Nethermind (Execution Client)
- Lighthouse (Consensus Client)
- systemd services
- JWT-authenticated Engine API

---

## 📍 Where This File Lives

Create this file at:

    nodes/operator-checklist.md

---

## 🛠️ How to Create This File (GitHub UI)

1. Go to your repo root
2. Click **Add file** → **Create new file**
3. In the filename field, type:

    nodes/operator-checklist.md

4. Paste this entire document
5. Click **Commit changes**

---

## 📁 Recommended Repo Structure

    blockchain-infra-playbook/
    ├── linux/
    ├── networking/
    ├── systemd/
    ├── nodes/
    │   └── operator-checklist.md
    ├── assets/
    └── README.md

---

## 🧑‍💻 Node Operator Checklist

This checklist helps monitor:

- system health
- node sync progress
- networking
- logs
- storage

---

## 🟢 Daily Health Check (2 to 5 minutes)

Run this once per day:

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager

    ss -tulpn | grep -E '8545|8551|30303|9000'

    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse

    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

### ✅ Verify

- both services show `active (running)`
- ports are open:
  - `8545` → JSON-RPC
  - `8551` → Engine API
  - `30303` → execution P2P
  - `9000` → consensus P2P
- disk usage is increasing over time
- logs show normal activity and no critical errors

---

## 👀 Live Monitoring (Optional)

Use when debugging or watching sync in real time:

    journalctl -u nethermind -f

    journalctl -u lighthouse-beacon -f

---

## 🟡 Every Few Days (5 to 10 minutes)

    df -h
    free -h

    ps aux | grep nethermind
    ps aux | grep lighthouse

### ✅ Verify

- enough disk space remains
- memory usage is stable
- both processes are still running

---

## 🔵 Weekly Deep Check (10 minutes)

    systemctl status nethermind --no-pager -l
    systemctl status lighthouse-beacon --no-pager -l

    journalctl -u nethermind -n 100 --no-pager
    journalctl -u lighthouse-beacon -n 100 --no-pager

    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse

### ✅ What to Look For

- no repeated crashes
- no restart loops
- no permission errors
- no JWT / Engine API authentication errors
- logs show continued syncing and block processing
- disk growth is consistent

---

## 🔧 Debugging Commands (When Something Breaks)

    journalctl -u nethermind -f
    journalctl -u lighthouse-beacon -f

    ss -tulpn | grep -E '8545|8551|30303|9000'

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_ok || echo nethermind_fail
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_ok || echo lighthouse_fail

### 🔁 Restart Services (if required)

    sudo systemctl restart nethermind
    sudo systemctl restart lighthouse-beacon

---

## 🧠 Healthy Node Indicators

- both services consistently show `active (running)`
- ports remain open and stable
- Nethermind disk usage steadily increases during sync
- Lighthouse continues showing new blocks / synced state
- logs show:
  - peer connections
  - block processing
  - forkchoice updates

---

## ⚠️ Warning Signs

Investigate if you see:

- `failed` service status
- repeated restarts
- `permission denied`
- JWT / Engine API authentication failures
- ports not listening
- disk usage stops growing unexpectedly
- excessive memory or swap usage

---

## 📌 Notes

- This node runs on Ubuntu under WSL2
- systemd manages startup and restart behavior
- JWT secret is required for execution ↔ consensus communication
- This checklist reflects real-world node operation practices

---

## 🚀 Goal

Build operational confidence managing blockchain infrastructure by learning how to:

- monitor node health
- debug issues quickly
- understand system behavior
- develop production-style operating habits

---

## 🔥 Quick Copy-Paste Daily Block

If you just want the short daily operator routine, run this:

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    ss -tulpn | grep -E '8545|8551|30303|9000'
    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse
    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

---

## 🔥 Quick Copy-Paste Weekly Block

    systemctl status nethermind --no-pager -l
    systemctl status lighthouse-beacon --no-pager -l
    journalctl -u nethermind -n 100 --no-pager
    journalctl -u lighthouse-beacon -n 100 --no-pager
    sudo du -sh /home/nethermind/data
    sudo du -sh /var/lib/lighthouse
    df -h
    free -h

---

## ✅ Summary

This file belongs at:

    nodes/operator-checklist.md

And it documents the real day-to-day workflow for operating a full Ethereum node with:

- Nethermind
- Lighthouse
- systemd
- JWT-secured Engine API
