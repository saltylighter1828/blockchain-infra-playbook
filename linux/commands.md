# 🐧 Linux Commands

A compact reference for core Linux commands used during Ethereum node operations, service validation, and system troubleshooting.

These commands are part of the day-to-day workflow for checking whether services are running, ports are listening, storage is healthy, and blockchain data is growing as expected.

---

## 🔍 Process Check

    ps aux | grep nethermind

Use this to confirm whether the Nethermind process is present.

### What it shows
- the running process
- the user it is running as
- CPU and memory usage
- the command used to start it

### Notes
- the `grep` line often appears in the output as well
- the important line is the actual Nethermind process
- useful for confirming that a service exists at the process level, even before deeper checks

### Common Use Cases
- confirm Nethermind is running
- inspect memory or CPU usage
- cross-check against `systemctl status`

---

## 🌐 Port Check

    ss -tulpn | grep -E '8545|8551|30303'

Use this to verify that key Nethermind ports are listening.

### What it checks
- `8545` → JSON-RPC
- `8551` → Engine API
- `30303` → Nethermind P2P networking

### Why it matters
- confirms the service is not only running, but listening on expected network ports
- helps detect startup, binding, or configuration issues
- useful when validating execution-consensus communication or external node access

### Notes
- `8545` is typically used for JSON-RPC queries
- `8551` is the Engine API used by Lighthouse to communicate with Nethermind
- `30303` is used for peer-to-peer network participation

### Common Use Cases
- verify Nethermind startup completed successfully
- confirm Engine API is listening
- check whether peer-to-peer networking is active

---

## 💾 Disk Space

    df -h

Use this to inspect available disk space across mounted filesystems.

### What it shows
- total size
- used space
- available space
- percentage used
- mount point

### Why it matters
- Ethereum node data grows over time
- low free space can slow the node or eventually cause service failures
- disk pressure is one of the most common operational risks for blockchain nodes

### Common Use Cases
- confirm there is enough space for continued sync
- monitor root filesystem health
- check storage headroom before large sync or growth events

### Practical Tip
For the most relevant quick check on the Linux root filesystem:

    df -h /

---

## 📦 Data Size

    du -sh /mnt/n/nethermind-data

Use this to check the current size of the Nethermind blockchain data directory.

### What it shows
- total disk usage of the directory in human-readable form

### Why it matters
- helps confirm that blockchain data is present and growing
- useful when tracking sync progress over time
- helps validate where node data is being stored

### Notes
- this repository uses Nethermind data at:

      /mnt/n/nethermind-data

- if your data directory changes, update the path accordingly

### Common Use Cases
- confirm the node is writing data
- compare growth over time
- validate storage location after configuration changes

---

## Recommended Companion Commands

These commands pair well with the checks above.

### Service Status

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager

Use these to confirm the service state at the `systemd` level.

### Recent Logs

    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

Use these to review recent activity and spot warnings or errors.

### Memory Check

    free -h

Use this to review RAM and swap usage.

---

## Healthy Looks Like

A healthy system usually shows:

- Nethermind process present
- expected ports listening
- sufficient disk space available
- blockchain data directory growing
- service status showing `active (running)`
- recent logs showing normal activity

---

## Warning Signs

Investigate further if you see:

- no Nethermind process
- missing expected ports
- high disk usage with little free space remaining
- no growth in node data over time
- repeated service restarts
- recent logs showing persistent errors

---

## Key Takeaway

These commands form part of the basic operator toolkit.

They are simple, but highly useful because they answer the first operational questions quickly:

- is the process running?
- are the expected ports open?
- is storage healthy?
- is the node actually writing data?

Used together, they provide a fast first-pass health check before moving into deeper debugging.
