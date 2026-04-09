# Nethermind Operator Checks on WSL Ubuntu

This file documents the commands I use to inspect, monitor, and verify the health of the Nethermind execution client running as a `systemd` service on Ubuntu under WSL2.

---

## Purpose

These checks help answer:

- Is the Nethermind service running?
- What do the recent logs say?
- Is the process alive?
- Are the expected ports listening?
- Is the data directory growing?
- Is the machine running out of disk space?
- Is the node waiting for a consensus client?

---

## 1. Check if the service is running

    sudo systemctl status nethermind --no-pager

What this tells me:
- whether the service is loaded
- whether it is active or failed
- the main process ID
- memory and CPU usage
- recent service log lines

Healthy sign:
- `Active: active (running)`

---

## 2. Follow live logs

    journalctl -u nethermind -f

What this tells me:
- live node activity
- recent warnings or errors
- whether the client is connecting to peers
- whether the execution client is waiting for the consensus layer

To exit live log view:

    Ctrl + C

---

## 3. View recent logs without live follow

    journalctl -u nethermind -n 50 --no-pager

What this tells me:
- recent events without entering the pager
- whether the service has restarted
- whether sync-related or networking-related messages are appearing

---

## 4. Check the running process

    ps aux | grep nethermind

What this tells me:
- whether the Nethermind process exists
- which user is running it
- the command used to launch it
- the process ID

Expected:
- the main process should be running under the `nethermind` user

---

## 5. Check listening ports

    ss -tulpn | grep -E '8545|8551|30303'

What this tells me:
- whether the expected ports are open
- whether RPC is local-only
- whether peer-to-peer networking is listening

Expected ports:
- `30303` = Ethereum P2P networking
- `8545` = JSON-RPC
- `8551` = execution to consensus engine API

Healthy sign:
- `8545` and `8551` listening on localhost
- `30303` listening for peer networking

---

## 6. Check total disk usage

    df -h

What this tells me:
- how much free disk space remains
- whether the WSL environment is approaching storage pressure
- which mounted drives are available

Why it matters:
- blockchain clients grow over time
- low disk space can cause crashes, corruption risk, or sync issues

---

## 7. Check Nethermind data directory growth

    sudo du -sh /home/nethermind/data

What this tells me:
- how large the Nethermind data directory is
- whether the node is writing data as expected

Why it matters:
- confirms the client is actually storing chain data
- useful for tracking growth over time

---

## 8. Restart the service

    sudo systemctl restart nethermind

What this does:
- restarts the service cleanly through `systemd`

Follow-up checks:
- run `sudo systemctl status nethermind --no-pager`
- run `journalctl -u nethermind -n 50 --no-pager`

---

## 9. Stop the service

    sudo systemctl stop nethermind

What this does:
- stops Nethermind cleanly

Why I might use it:
- before moving the distro
- before maintenance
- before changing configuration
- before troubleshooting startup behavior

---

## 10. Start the service

    sudo systemctl start nethermind

What this does:
- starts the Nethermind service again after it has been stopped

---

## 11. Common observations in my current setup

At this stage, the logs show that Nethermind is waiting for forkchoice messages from a consensus client.

This means:
- the execution client is running correctly
- peer networking is active
- the node is not yet a complete post-merge Ethereum node
- a consensus client still needs to be added later

Typical log pattern:

    Waiting for Forkchoice message from Consensus Layer
    Not receiving ForkChoices from the consensus client that are required to sync.

This is expected while running Nethermind on its own.

---

## 12. My quick health checklist

When checking the node, I look for these signs:

- `systemctl status` shows `active (running)`
- recent logs do not show repeated crashes
- `ps aux` shows the Nethermind process alive
- `ss -tulpn` shows `30303`, `8545`, and `8551`
- `du -sh /home/nethermind/data` shows the data directory exists and grows
- `df -h` shows enough free disk space remains

---

## 13. Command cheat sheet

    sudo systemctl status nethermind --no-pager
    journalctl -u nethermind -f
    journalctl -u nethermind -n 50 --no-pager
    ps aux | grep nethermind
    ss -tulpn | grep -E '8545|8551|30303'
    df -h
    sudo du -sh /home/nethermind/data
    sudo systemctl restart nethermind
    sudo systemctl stop nethermind
    sudo systemctl start nethermind

---

## 14. What I learned

- how to inspect a `systemd` service
- how to read recent logs with `journalctl`
- how to follow live logs
- how to confirm a process is running
- how to verify ports are listening
- how to track data directory growth
- how to think about storage and persistence
- how to recognize that an execution client alone still needs a consensus client

---

## 15. Outcome

I can now:
- install and manage Nethermind as a Linux service
- inspect service health with `systemctl`
- inspect logs with `journalctl`
- verify ports and processes
- monitor storage and data growth
- explain the difference between execution-client health and full-node completeness
