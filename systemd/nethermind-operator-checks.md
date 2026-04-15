# Nethermind Operator Checks on Ubuntu WSL2

This document captures the practical checks used to inspect, monitor, and verify the health of the Nethermind execution client running as a `systemd` service on Ubuntu under WSL2.

It is intended as a compact operator reference for day-to-day validation, troubleshooting, and service control. Rather than focusing only on whether the process exists, these checks are designed to confirm service health across multiple layers: service state, logs, process visibility, network ports, storage growth, and host capacity. :contentReference[oaicite:0]{index=0}

---

## Purpose

These checks help answer a small set of operational questions quickly:

- Is the Nethermind service running correctly?
- What do the recent logs indicate?
- Is the process alive at the OS level?
- Are the expected ports listening?
- Is the data directory growing as expected?
- Is the machine under disk pressure?
- Is Nethermind behaving like a healthy execution client?

In practice, this file acts as a short runbook for validating Nethermind before moving into deeper debugging.

---

## Environment Context

Current environment:

- Host OS: Windows 11
- Linux environment: Ubuntu on WSL2
- Service manager: `systemd`
- Execution client: Nethermind
- Current Nethermind data directory: `/mnt/n/nethermind-data`

---

## 1. Check Service Status

    sudo systemctl status nethermind --no-pager

### What this shows
- whether the service is loaded
- whether it is active, inactive, or failed
- the main process ID
- CPU and memory usage
- recent service log lines

### Healthy sign
- `Active: active (running)`

### Why it matters
This is the fastest single command for checking whether Nethermind is running properly under `systemd`.

---

## 2. Follow Live Logs

    journalctl -u nethermind -f

### What this shows
- live node activity
- recent warnings or errors
- peer activity and sync behaviour
- interaction with the consensus layer once connected
- block processing and forkchoice-related messages

### Exit live view

    Ctrl + C

### Why it matters
This is the best command for watching real-time behaviour during startup, restart, sync, or troubleshooting.

---

## 3. View Recent Logs Without Live Follow

    journalctl -u nethermind -n 50 --no-pager

### What this shows
- the latest service events without entering interactive pager mode
- whether the service recently restarted
- whether sync, networking, pruning, or execution messages are appearing
- short-term health signals without needing a live stream

### Why it matters
This is useful when you want recent context quickly without staying attached to the service.

---

## 4. Check the Running Process

    ps aux | grep nethermind

### What this shows
- whether the Nethermind process exists
- which user is running it
- the launch command
- the process ID
- rough CPU and memory usage at the process level

### Expected result
- the main process should appear under the `nethermind` user
- a `grep` line may appear as well and can be ignored

### Why it matters
This confirms the process exists independently of `systemd` output and provides a quick OS-level view of runtime behaviour.

---

## 5. Check Listening Ports

    ss -tulpn | grep -E '8545|8551|30303|6060'

### What this shows
- whether the expected Nethermind ports are open
- whether JSON-RPC and Engine API are local-only
- whether peer-to-peer networking is listening
- whether Nethermind metrics are exposed

### Expected ports
- `30303` → Ethereum execution-layer peer-to-peer networking
- `8545` → JSON-RPC
- `8551` → Engine API
- `6060` → Nethermind metrics

### Healthy sign
- `8545` and `8551` listening on localhost
- `30303` listening for peer networking
- `6060` listening if metrics are enabled

### Why it matters
A service can be running but still not be listening on the interfaces or ports you expect. This helps confirm runtime readiness, not just process existence.

---

## 6. Check Total Disk Usage

    df -h
    df -h /

### What this shows
- available disk space
- used space
- percentage used
- mounted filesystems
- remaining space on the Linux root filesystem

### Why it matters
Ethereum clients grow over time. Low disk space can lead to degraded performance, failed writes, sync issues, or service instability.

### Operational focus
The most important quick check is usually:

    df -h /

because it tells you whether the main Linux environment still has safe free-space headroom.

---

## 7. Check Nethermind Data Directory Growth

    sudo du -sh /mnt/n/nethermind-data

### What this shows
- total size of the Nethermind data directory
- whether the node is writing data as expected
- how storage usage changes over time

### Why it matters
This is a simple but useful signal that the execution client is storing chain data and continuing to build its local state.

### Note
If the data path changes in future, update this command accordingly.

---

## 8. Restart the Service

    sudo systemctl restart nethermind

### What this does
- restarts the Nethermind service cleanly through `systemd`

### Recommended follow-up
After restarting, verify recovery with:

    sudo systemctl status nethermind --no-pager
    journalctl -u nethermind -n 50 --no-pager
    ss -tulpn | grep -E '8545|8551|30303|6060'

### Why it matters
A restart is only useful if recovery is verified afterward. The goal is not just to bounce the service, but to confirm that it came back healthy.

---

## 9. Stop the Service

    sudo systemctl stop nethermind

### What this does
- stops Nethermind cleanly through `systemd`

### Common reasons to stop it
- before maintenance
- before configuration changes
- before moving data
- before controlled shutdown of the environment
- before troubleshooting startup behaviour

### Why it matters
Controlled shutdown is safer than killing processes manually and keeps service state cleaner.

---

## 10. Start the Service

    sudo systemctl start nethermind

### What this does
- starts the Nethermind service after it has been stopped

### Recommended follow-up
After starting, confirm:

    sudo systemctl status nethermind --no-pager
    journalctl -u nethermind -n 50 --no-pager

---

## 11. Common Behaviour in Execution-Only Mode

When Nethermind is running without a consensus client, logs may show that it is waiting for forkchoice messages.

Typical log patterns include:

    Waiting for Forkchoice message from Consensus Layer
    Not receiving ForkChoices from the consensus client that are required to sync.

### What this means
- the execution client is running correctly
- peer networking may still be active
- the service itself is not necessarily broken
- the node is not yet a complete post-Merge Ethereum node
- a consensus client is still required for full execution-consensus operation

### Why it matters
This is an important distinction: execution-client health is not the same thing as full-node completeness.

---

## 12. Quick Health Checklist

When validating Nethermind, the main things to look for are:

- `systemctl status` shows `active (running)`
- recent logs do not show repeated crash loops
- `ps aux` shows the Nethermind process alive
- `ss -tulpn` shows `30303`, `8545`, `8551`, and ideally `6060` if metrics are enabled
- `du -sh /mnt/n/nethermind-data` shows the data directory exists and grows over time
- `df -h` shows sufficient free disk space remains

If those checks look normal together, Nethermind is usually in a healthy execution-layer state.

---

## 13. Command Cheat Sheet

    sudo systemctl status nethermind --no-pager
    journalctl -u nethermind -f
    journalctl -u nethermind -n 50 --no-pager
    ps aux | grep nethermind
    ss -tulpn | grep -E '8545|8551|30303|6060'
    df -h
    df -h /
    sudo du -sh /mnt/n/nethermind-data
    sudo systemctl restart nethermind
    sudo systemctl stop nethermind
    sudo systemctl start nethermind

---

## 14. What This Teaches

These checks build practical understanding of:

- how to inspect a `systemd` service
- how to use `journalctl` for recent and live logs
- how to confirm a process exists at the OS level
- how to verify expected ports are listening
- how to monitor data growth and persistence
- how to think about storage pressure in node environments
- how to distinguish a healthy execution client from a complete post-Merge node

---

## 15. Outcome

With these checks in place, Nethermind can be operated more confidently as a Linux service.

This means being able to:

- inspect service health with `systemctl`
- inspect logs with `journalctl`
- verify runtime ports and processes
- monitor data growth and storage safety
- restart or stop the service in a controlled way
- explain the difference between execution-layer health and full-node completeness

Together, these commands form a solid first-line operator toolkit for Nethermind on Ubuntu under WSL2.
