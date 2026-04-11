# Grafana Monitoring Dashboard

Built a Grafana dashboard using Prometheus and node_exporter to monitor the Ethereum node host on WSL2. What started as a basic dashboard has now been expanded into a more structured host-monitoring view with separate sections for host health, storage, and network activity.

The goal was to move beyond simple “is it running?” checks and build a dashboard that helps spot system behaviour, resource pressure, and storage trends before they become incidents.

---

## Current Dashboard Sections

### Host Health

Tracks core machine health and scrape availability:

- **Memory Usage %**  
  Shows RAM usage over time.

- **CPU Usage %**  
  Shows estimated CPU busy percentage over time.

- **Root Disk Used %**  
  Shows how full the root filesystem is.

- **Load Average**  
  Displays Linux load averages over 1 minute, 5 minutes, and 15 minutes to show short-term and long-term system pressure.

- **System Uptime**  
  Shows how long the host has been running since the last reboot.

- **Prometheus Up**  
  Confirms Prometheus is being scraped successfully.

- **Node Exporter Up**  
  Confirms node_exporter is being scraped successfully.

### Storage

Tracks both current storage state and storage behaviour over time:

- **Root Disk Used % Over Time**  
  Shows root filesystem growth as a percentage.

- **Root Free Space Over Time**  
  Shows available free space on the root filesystem over time.

- **Root Disk Used %**  
  Quick stat for current root disk usage.

- **Root Free Space**  
  Quick stat for current available root free space.

### Network

Tracks host traffic activity:

- **Network Receive**  
  Shows inbound throughput over time.

- **Network Transmit**  
  Shows outbound throughput over time.

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

---

## What I Learned

- `up = 1` means Prometheus can successfully scrape that target.
- Host monitoring becomes much more useful when panels are grouped by purpose instead of mixed together.
- CPU %, memory %, and load average each tell a different story about system pressure.
- Load average is not the same as CPU usage. It shows how much work is running or waiting over different time windows.
- Tracking both current storage values and storage trends over time makes it easier to spot unhealthy growth patterns early.
- `node_filesystem_avail_bytes` is helpful for free-space visibility, while used percentage is often easier to read quickly during troubleshooting.
- WSL-mounted Windows drives exposed through `9p` and `drvfs` do not behave as cleanly in node_exporter as native Linux filesystems like `ext4`, so the dashboard was kept focused on the root Linux filesystem for reliable storage monitoring.

---

## Current Observations

- Prometheus and node_exporter are both up and being scraped successfully.
- Memory usage is active but stable overall.
- CPU usage shows normal activity without sustained high pressure.
- Load average shows short bursts but no major long-duration stress.
- Root disk usage is low overall, with free space around the 940 GiB range.
- The storage graphs show a repeating sawtooth pattern, which suggests normal write activity followed by periodic cleanup or space recovery.
- Network receive and transmit panels confirm the host is actively sending and receiving traffic.

---

## Why This Matters

This dashboard is the foundation for moving from reactive troubleshooting to proactive monitoring.

After working through storage-path issues, service checks, and general node debugging, building this Grafana view helped turn those lessons into something operationally useful. It gives a fast visual check of host health and makes it easier to notice patterns in disk growth, system load, and network activity.

---

## Screenshots

### Host Health Dashboard

![Host Health Dashboard](../assets/screenshots/monitoring/grafana-host-health-dashboard.png)

### Storage and Network Dashboard

![Storage and Network Dashboard](../assets/screenshots/monitoring/grafana-storage-network-dashboard.png)

---

## Next Step

The next upgrade is to extend this from **host monitoring** into **Ethereum client monitoring** by exposing and scraping:

- **Nethermind metrics**
- **Lighthouse metrics**

That will allow future dashboard panels for:

- Peer count
- Sync status
- Head progress
- Client-specific health
