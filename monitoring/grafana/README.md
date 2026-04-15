# Grafana Monitoring Dashboard

This dashboard was built to monitor a live Ethereum node stack running on Ubuntu under WSL2 using Prometheus, Node Exporter, Grafana, Nethermind, and Lighthouse.

What began as a simple host-health dashboard has developed into a broader operator-facing monitoring view covering host health, storage, network activity, Lighthouse consensus monitoring, and Nethermind execution monitoring. The focus is not only on confirming that services are running, but on making system behaviour visible early enough to detect pressure, lag, or unhealthy patterns before they turn into incidents. :contentReference[oaicite:0]{index=0}

---

## Purpose

The goal of this dashboard is to move from basic service checks toward practical, layered observability for Ethereum infrastructure.

It is designed to answer questions such as:

- Is the host healthy enough to support the node?
- Are Prometheus targets being scraped successfully?
- Is Lighthouse synced and keeping up?
- Is Nethermind tracking the chain head cleanly?
- Is storage, disk I/O, or network behaviour showing early warning signs?

In that sense, the dashboard acts as an operational view of the full stack rather than a collection of unrelated charts. :contentReference[oaicite:1]{index=1}

---

## Current Dashboard Coverage

### Host Health

Tracks baseline machine condition and scrape availability.

Panels include:

- **Memory Usage %**  
  Shows RAM usage over time.

- **CPU Usage %**  
  Shows estimated CPU busy percentage over time.

- **Root Disk Used %**  
  Shows how full the root filesystem is.

- **Load Average**  
  Displays Linux load averages over 1 minute, 5 minutes, and 15 minutes to show short-term and sustained system pressure.

- **System Uptime**  
  Shows how long the host has been running since the last reboot.

- **Prometheus Up**  
  Confirms the Prometheus target is being scraped successfully.

- **Node Exporter Up**  
  Confirms the Node Exporter target is being scraped successfully.

### Network

Tracks host traffic activity.

Panels include:

- **Network Receive**  
  Shows inbound throughput over time.

- **Network Transmit**  
  Shows outbound throughput over time.

### Storage

Tracks both current storage state and storage behaviour over time.

Panels include:

- **Root Disk Used % Over Time**  
  Shows filesystem growth as a percentage.

- **Root Free Space Over Time**  
  Shows available free space over time.

- **Disk Read**  
  Shows disk read throughput over time for the active block device.

- **Disk Write**  
  Shows disk write throughput over time for the active block device.

- **Root Disk Used %**  
  Quick stat for current root disk usage.

- **Root Free Space**  
  Quick stat for current available root free space.

### Lighthouse

Tracks consensus-client health and sync behaviour.

Panels include:

- **Lighthouse Up**  
  Confirms the Lighthouse metrics target is being scraped successfully.

- **Synced Status**  
  Shows whether Lighthouse reports itself as synced.

- **Lighthouse Peers**  
  Shows current beacon-node peer count.

- **Head Slot**  
  Shows the latest beacon-chain head slot known to Lighthouse.

- **Finalized Epoch**  
  Shows the most recently finalized epoch.

- **Current Epoch**  
  Shows the present epoch according to the beacon-chain slot clock.

- **Slot Lag**  
  Shows the difference between the present slot and Lighthouse’s head slot, helping confirm whether the node is keeping up.

- **Sync Slots/sec**  
  Shows Lighthouse sync speed during catch-up activity.

- **Head Slot Rate**  
  Shows the rate of head-slot progression over time.

### Nethermind

Tracks execution-client health, sync state, peer activity, execution progress, and processing behaviour.

Panels include:

- **Nethermind Up**  
  Confirms the Nethermind metrics target is being scraped successfully.

- **Sync Status**  
  Shows whether Nethermind reports itself as syncing or fully synced.

- **Peers**  
  Shows the current connected peer count.

- **Sync Peers**  
  Shows how many peers are actively helping with sync.

- **Block Lag**  
  Shows the difference between the best known block and the local chain height.

- **Blocks Observed (5m)**  
  Shows approximate block activity over the last 5 minutes.

- **Chain Height**  
  Shows the current local blockchain height known to Nethermind.

- **Best Known Block**  
  Shows the best block number currently known from peers and network state.

- **Block Processing Time**  
  Shows how long Nethermind took to process the most recent block, in milliseconds. :contentReference[oaicite:2]{index=2}

---

## Key Queries Used

### Prometheus target health

    up{job="prometheus"}

    up{job="node"}

### CPU usage %

    100 * (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance))

### Memory usage %

    100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))

### Root disk used %

    100 * (1 - (node_filesystem_avail_bytes{mountpoint="/",fstype="ext4"} / node_filesystem_size_bytes{mountpoint="/",fstype="ext4"}))

### Root free space

    node_filesystem_avail_bytes{mountpoint="/",fstype="ext4"}

### System uptime

    node_time_seconds - node_boot_time_seconds

### Load average

    node_load1

    node_load5

    node_load15

### Network receive

    rate(node_network_receive_bytes_total{device!="lo"}[5m])

### Network transmit

    rate(node_network_transmit_bytes_total{device!="lo"}[5m])

### Disk read throughput

    rate(node_disk_read_bytes_total{device="sdd"}[5m])

### Disk write throughput

    rate(node_disk_written_bytes_total{device="sdd"}[5m])

### Lighthouse up

    up{job="lighthouse"}

### Lighthouse synced status

    sync_eth2_synced{job="lighthouse"}

### Lighthouse peers

    libp2p_peers{job="lighthouse"}

### Lighthouse head slot

    beacon_head_slot{job="lighthouse"}

### Lighthouse finalized epoch

    beacon_finalized_epoch{job="lighthouse"}

### Lighthouse current epoch

    slotclock_present_epoch{job="lighthouse"}

### Lighthouse slot lag

    slotclock_present_slot{job="lighthouse"} - beacon_head_slot{job="lighthouse"}

### Lighthouse sync slots/sec

    sync_slots_per_second{job="lighthouse"}

### Lighthouse head slot rate

    rate(beacon_head_slot{job="lighthouse"}[5m])

### Nethermind up

    up{job="nethermind"}

### Nethermind peers

    sum(ethereum_peer_count)

### Nethermind sync peers

    sum(nethermind_sync_peers)

### Nethermind sync status

    max(nethermind_state_synced)

### Nethermind chain height

    max(ethereum_blockchain_height)

### Nethermind best known block

    max(ethereum_best_known_block_number)

### Nethermind block lag

    max(ethereum_best_known_block_number) - max(ethereum_blockchain_height)

### Nethermind blocks observed (5m)

    increase(nethermind_blocks[5m])

### Nethermind block processing time

    max(nethermind_last_block_processing_time_in_ms)

---

## What This Dashboard Helps Validate

This dashboard makes it easier to validate the node stack across multiple operational layers at once.

### Host Layer
- whether CPU, memory, load, and disk look stable
- whether storage is filling unexpectedly
- whether disk activity looks normal

### Monitoring Layer
- whether Prometheus and Node Exporter are both up
- whether metrics are being scraped successfully
- whether key queries remain usable

### Consensus Layer
- whether Lighthouse is up
- whether it reports itself as synced
- whether peer count and head-slot progression look healthy
- whether slot lag remains low

### Execution Layer
- whether Nethermind is up
- whether it is synced
- whether chain height is tracking the best known block
- whether block lag remains near zero
- whether block processing time remains low and stable

This turns the dashboard into a practical validation surface rather than a purely visual summary. :contentReference[oaicite:3]{index=3}

---

## Key Lessons Learned

A few important operational takeaways emerged while building and refining the dashboard:

- `up = 1` means Prometheus can successfully scrape a target
- host monitoring is much easier to interpret when panels are grouped by operational purpose
- CPU usage, memory usage, and load average each describe different forms of pressure
- load average is not the same as CPU usage; it reflects work running or waiting over time
- storage monitoring becomes more useful when both current values and long-term trends are visible
- `node_filesystem_avail_bytes` is useful for raw free-space tracking, while used percentage is often faster to interpret at a glance
- WSL-mounted Windows filesystems exposed via `9p` and `drvfs` are less reliable for clean Linux-style storage monitoring, so the dashboard focuses on the native Linux root filesystem where appropriate
- disk I/O adds an important performance layer because a node can appear healthy on CPU while still being limited by storage behaviour
- Lighthouse metrics are not always named with a `lighthouse_` prefix, so discovering the actual metric names through Prometheus exploration was an important part of the build
- `sync_eth2_synced` is a much stronger sync signal than trying to infer sync visually
- `slotclock_present_slot - beacon_head_slot` is a simple and useful consensus lag indicator
- a small slot lag of `0` to `1` is normal during steady-state operation
- `rate(beacon_head_slot[5m])` works well as a heartbeat-style signal for beacon-chain progression
- `sync_slots_per_second` is most useful during catch-up and recovery rather than as a steady-state health metric
- Nethermind metrics often expose multiple series, so `sum()` and `max()` are helpful for producing cleaner operator panels
- `ethereum_blockchain_height` reflects local execution progress, while `ethereum_best_known_block_number` reflects the best network head known from peers
- block lag is a simple and effective execution-client health signal
- a lag of `0` or near `0` usually indicates Nethermind is keeping up
- lower block processing time is generally better because it reflects faster block handling
- `increase(nethermind_blocks[5m])` is a useful execution-layer activity signal

These lessons helped shape the dashboard from a collection of metrics into a more usable operational tool. :contentReference[oaicite:4]{index=4}

---

## Current Observations

Based on the current dashboard state:

- Prometheus and Node Exporter are up and scraping successfully
- memory usage is active but stable overall
- CPU usage shows normal activity without sustained high pressure
- load average shows short bursts but no major long-duration stress
- root disk usage remains low, with large free-space headroom
- storage panels show normal write activity and gradual filesystem growth
- disk read and write panels confirm active Linux filesystem usage at the block-device level
- network receive and transmit panels confirm active host traffic
- Lighthouse is up and scraped successfully
- Lighthouse reports itself as synced
- Lighthouse peer count reflects active beacon-network connectivity
- head slot, current epoch, and finalized epoch are progressing as expected
- slot lag remains very low, indicating Lighthouse is keeping up with current chain state
- head slot rate is consistent with normal beacon-chain progression
- Nethermind is up and exposing metrics successfully
- Nethermind peer count and sync-peer count confirm active execution-layer connectivity
- chain height and best known block remain closely aligned
- block lag remains at or near zero, indicating healthy execution progress
- block processing time remains low, suggesting block import is being handled comfortably
- Blocks Observed (5m) acts as a useful heartbeat-style signal for ongoing execution activity

---

## Why This Matters

This dashboard is an important step in moving from reactive troubleshooting toward proactive monitoring.

After working through storage-path issues, service validation, JWT permission problems, metrics exposure, Prometheus scraping, Lighthouse monitoring, Nethermind monitoring, disk visibility, and general node debugging, building this Grafana view turned those lessons into something operationally useful.

Instead of checking one command at a time, the dashboard now provides a faster, more operator-friendly view of the stack across:

- host health
- storage behaviour
- disk activity
- network traffic
- consensus sync behaviour
- execution-client progress

That shift matters because it reduces guesswork and makes it easier to spot abnormal behaviour early. :contentReference[oaicite:5]{index=5}

---

## Screenshots

### Nethermind and Lighthouse Dashboard Overview

![Nethermind and Lighthouse Dashboard Overview](../../assets/screenshots/monitoring/Nethermind-and-lighthouse-dashboard-overview.png)

### Nethermind Dashboard Overview

![Nethermind Dashboard Overview](../../assets/screenshots/monitoring/Nethermind-dashboard-overview.png)

### Lighthouse Dashboard Overview

![Lighthouse Dashboard Overview](../../assets/screenshots/monitoring/lighthouse-dashboard-overview.png)

### Host Health Dashboard

![Host Health Dashboard](../../assets/screenshots/monitoring/grafana-host-health-dashboard.png)

### Network Dashboard

![Network Dashboard](../../assets/screenshots/monitoring/grafana-network-dashboard.png)

### Storage Dashboard

![Storage Dashboard](../../assets/screenshots/monitoring/grafana-storage-dashboard.png)

---

## Next Step

The next upgrade path is not simply adding more raw panels, but increasing operational maturity around the existing dashboard.

Good next steps include:

- basic alerting for service-down, high block lag, or low disk space
- practicing restart and recovery validation workflows against the dashboard
- learning clearer normal ranges for peers, lag, processing time, CPU, memory, and disk I/O
- refining panel layout and documentation as the stack matures

This dashboard is now a solid v1 monitoring foundation for an Ethereum node across the host, consensus layer, and execution layer.
