# Prometheus + Node Exporter + Grafana Setup on Ubuntu WSL2 for Ethereum Node Monitoring

This document records the full end-to-end setup and validation of a local observability stack for Ethereum node infrastructure on Ubuntu under WSL2.

It covers installation and verification of:

- **Node Exporter**
- **Prometheus**
- **Grafana**

alongside an existing Ethereum node stack using:

- **Nethermind**
- **Lighthouse**
- **systemd**

It also captures the real issues encountered during setup, including:

- swap usage interpretation
- WSL memory pressure
- increasing WSL resources with `.wslconfig`
- browser access issues from Windows to WSL services
- Prometheus endpoint validation
- Grafana login confusion
- confirming end-to-end monitoring works

---

## Purpose

The goal of this setup is to establish a practical monitoring foundation for blockchain infrastructure.

### Component Roles

- **Node Exporter** exposes Linux host metrics such as CPU, memory, disk, and network
- **Prometheus** scrapes and stores those metrics over time
- **Grafana** visualizes the data through dashboards and queries

Together, they provide a production-style observability layer for learning and operating infrastructure with better visibility.

---

## Environment

- Windows host
- Ubuntu on WSL2
- `systemd` enabled
- Ethereum node services already running:
  - Nethermind
  - Lighthouse

---

## Step 1: Validate Existing Ethereum Node Services

Before installing monitoring tools, both Ethereum services were checked to confirm the node stack was already healthy.

### Check Lighthouse

    systemctl status lighthouse-beacon --no-pager

Healthy indicators:

- `Loaded: loaded`
- `Active: active (running)`
- logs showing activity such as:
  - `New block received`
  - `Synced`
  - `Downloading historical blocks`

### Check Nethermind

    systemctl status nethermind --no-pager

Healthy indicators:

- `Loaded: loaded`
- `Active: active (running)`
- Engine API calls visible such as:
  - `engine_forkchoiceUpdatedV3`
  - `engine_newPayloadV4`
  - `engine_getBlobsV2`
- no obvious errors

### Key Realization

Once both services were confirmed as `systemd` units, the original terminal tab used for syncing was no longer required. Closing the terminal would not stop the services because `systemd` was managing them in the background.

---

## Step 2: Understand the Memory and Swap Problem

Memory and swap usage were checked with:

    free -h
    df -h

Initial results looked roughly like this:

- around `15 GiB` RAM available to WSL
- around `2.5 GiB` swap in use

### What Swap Means

Swap is overflow memory on disk. When RAM becomes crowded, Linux moves less-active memory out of RAM and into swap.

A simple analogy:

- **RAM** = workbench
- **Swap** = boxes on the floor

Swap is slower than RAM. A small amount of swap use is not unusual, but sustained or heavy swap use can reduce performance.

### What Looked Concerning

The node services were healthy, but:

- Nethermind was using significant RAM
- Lighthouse was also using several GiB
- swap was already being used heavily

This suggested the problem was not broken node software, but insufficient WSL resource allocation.

---

## Step 3: Increase WSL RAM with `.wslconfig`

To reduce swap pressure and give the node more breathing room, WSL resource limits were increased on the Windows side.

### Important Note

This requires restarting WSL, which temporarily stops all running Linux services, including Nethermind and Lighthouse.

### Create `.wslconfig` on Windows

The config file was created at:

    C:\Users\Anton\.wslconfig

Contents:

    [wsl2]
    memory=24GB
    processors=8
    swap=8GB

### Save Correctly

Using Notepad was fine, but the file had to be saved as:

    .wslconfig

and **not**:

    .wslconfig.txt

To do that:

- open Notepad
- paste the config
- click **File > Save As**
- file name: `.wslconfig`
- save as type: **All Files**
- location: `C:\Users\Anton\`

### Restart WSL

From Windows PowerShell:

    wsl --shutdown

Then reopen Ubuntu.

### Verify New Limits Inside WSL

    free -h
    nproc
    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager

After the change, the result improved substantially:

- around `23 GiB` RAM visible to WSL
- around `19 GiB` available
- `0B` swap in use
- `8` CPU threads visible
- both services restarted cleanly under `systemd`

This confirmed the memory pressure issue came from WSL resource limits, not from broken node behaviour.

---

## Step 4: Install Node Exporter

Node Exporter was installed first because it exposes host-level machine metrics.

### Install and Start Node Exporter

    sudo apt update
    sudo apt install -y prometheus-node-exporter
    sudo systemctl enable --now prometheus-node-exporter

### Validate Node Exporter

    systemctl status prometheus-node-exporter --no-pager
    ss -tulpn | grep 9100
    curl http://localhost:9100/metrics | head

Healthy indicators:

- service `active (running)`
- port `9100` listening
- metrics text returned by `curl`

### Harmless Warning Encountered

Running:

    curl http://localhost:9100/metrics | head

produced a final line like:

    curl: (23) Failure writing output to destination

This was harmless. It happened because `head` stopped reading after the first few lines and closed the pipe early.

---

## Step 5: Install Prometheus

Prometheus was installed next to scrape Node Exporter and store metrics over time.

### Install and Start Prometheus

    sudo apt install -y prometheus
    sudo systemctl enable --now prometheus

### Validate Prometheus

    systemctl status prometheus --no-pager
    ss -tulpn | grep 9090

Healthy indicators:

- service `active (running)`
- port `9090` listening

---

## Step 6: Configure Prometheus to Scrape Node Exporter

Prometheus needed to be configured to scrape Node Exporter on port `9100`.

### Back Up the Config

    sudo cp /etc/prometheus/prometheus.yml /etc/prometheus/prometheus.yml.bak

### Edit the Config

    sudo nano /etc/prometheus/prometheus.yml

Final relevant config:

    global:
      scrape_interval: 15s
      evaluation_interval: 15s

      external_labels:
        monitor: 'example'

    alerting:
      alertmanagers:
        - static_configs:
            - targets: ['localhost:9093']

    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    scrape_configs:
      - job_name: 'prometheus'
        scrape_interval: 5s
        scrape_timeout: 5s
        static_configs:
          - targets: ['localhost:9090']

      - job_name: node
        static_configs:
          - targets: ['localhost:9100']

### Restart Prometheus

    sudo systemctl restart prometheus
    systemctl status prometheus --no-pager

---

## Step 7: Validate Prometheus Health and Scrape Targets

Prometheus looked odd for a moment right after restart, showing a short-lived strange status display. This turned out not to be a real problem.

### Full Validation Commands

    systemctl status prometheus --no-pager -l
    journalctl -u prometheus -n 50 --no-pager
    ss -tulpn | grep 9090
    curl http://127.0.0.1:9090/-/healthy

Healthy result:

    Prometheus Server is Healthy.

### Important Discovery About `/targets`

Trying:

    curl http://127.0.0.1:9090/targets | head

returned:

    404 page not found

This was not a broken server. The problem was using the wrong endpoint for terminal or API use.

### Correct API Endpoints

Use:

    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

Expected healthy result:

- `job="prometheus"` with `"health":"up"`
- `job="node"` with `"health":"up"`
- `up = 1` for both targets

This confirmed:

- Prometheus was healthy
- Node Exporter was healthy
- Prometheus was scraping Node Exporter successfully

---

## Step 8: Browser Access Problem from Windows

Trying to open Prometheus in a Windows browser using:

    http://localhost:9090/targets

and then:

    http://127.0.0.1:9090/targets

did not work.

### What This Meant

Prometheus was healthy inside WSL, but Windows browser access to WSL localhost was not working as expected.

### Confirm Prometheus Inside WSL

    curl http://127.0.0.1:9090/-/healthy

This worked, so Prometheus itself was healthy.

### Get the WSL IP

    hostname -I

Example result:

    172.26.168.66

That WSL IP was then used for browser access when needed.

---

## Step 9: Install Grafana

Grafana was installed to visualize Prometheus metrics.

### Install Grafana Repository and Package

    sudo apt-get install -y apt-transport-https wget gnupg
    sudo mkdir -p /etc/apt/keyrings
    sudo wget -O /etc/apt/keyrings/grafana.asc https://apt.grafana.com/gpg-full.key
    sudo chmod 644 /etc/apt/keyrings/grafana.asc
    echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
    sudo apt-get update
    sudo apt-get install -y grafana
    sudo systemctl daemon-reload
    sudo systemctl enable --now grafana-server

### Validate Grafana

    systemctl status grafana-server --no-pager -l
    journalctl -u grafana-server -n 50 --no-pager
    ss -tulpn | grep 3000
    curl http://127.0.0.1:3000/api/health

Healthy response:

    {
      "database": "ok",
      "version": "12.4.2",
      "commit": "ebade4c739e1aface4ce094934ad85374887a680"
    }

This confirmed Grafana was healthy and listening on port `3000`.

---

## Step 10: Grafana Login Confusion

Opening Grafana in the browser showed a Grafana sign-in page.

At first this looked like it might require a cloud account, but that was incorrect.

### Important Realization

This was the **local Grafana instance**, not Grafana Cloud.

### Correct Login

- username: `admin`
- password: `admin`

After login, Grafana prompts for a password change.

### URL Used

Since localhost forwarding was unreliable, Grafana was accessed using the WSL IP:

    http://172.26.168.66:3000

---

## Step 11: Add Prometheus as a Grafana Data Source

After logging into Grafana:

- go to **Connections**
- go to **Data sources**
- click **Add data source**
- choose **Prometheus**

### Prometheus Data Source Settings

- **Name**: `prometheus`
- **Prometheus server URL**: `http://172.26.168.66:9090`
- **Authentication**: `No Authentication`

No additional changes were needed.

### Save and Test

Grafana returned:

    Successfully queried the Prometheus API.

This confirmed Grafana could communicate with Prometheus successfully.

---

## Step 12: Validate End-to-End Monitoring in Grafana

To prove the full monitoring path was working:

- open **Explore**
- select the Prometheus data source
- use **Code** mode or enter a metric directly

### Test Query

    up

Healthy result:

    up{instance="localhost:9090", job="prometheus"} 1
    up{instance="localhost:9100", job="node"} 1

This confirmed the full monitoring path:

- Node Exporter exposing metrics
- Prometheus scraping metrics
- Grafana querying Prometheus

---

## Useful Validation Commands

### Check Ethereum Node Services

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

### Check WSL Memory and Disk

    free -h
    df -h
    nproc

### Check Node Exporter

    systemctl status prometheus-node-exporter --no-pager
    ss -tulpn | grep 9100
    curl http://127.0.0.1:9100/metrics | head

### Check Prometheus

    systemctl status prometheus --no-pager
    journalctl -u prometheus -n 50 --no-pager
    ss -tulpn | grep 9090
    curl http://127.0.0.1:9090/-/healthy
    curl -s http://127.0.0.1:9090/api/v1/targets | grep -E '"health"|"job"'
    curl -s "http://127.0.0.1:9090/api/v1/query?query=up"

### Check Grafana

    systemctl status grafana-server --no-pager
    journalctl -u grafana-server -n 50 --no-pager
    ss -tulpn | grep 3000
    curl http://127.0.0.1:3000/api/health

### Get WSL IP for Browser Access

    hostname -I

---

## Real Troubleshooting Summary

### 1. Lighthouse Syncing in Terminal

**Concern**

- worried that closing the original WSL tab would stop Lighthouse

**Resolution**

- checked `systemctl status lighthouse-beacon`
- confirmed Lighthouse was already running as a `systemd` service
- repeated the same check for Nethermind

**Conclusion**

- closing the original syncing tab was safe

### 2. High Swap Usage

**Concern**

- swap usage seemed large and potentially unhealthy

**Resolution**

- learned that swap is overflow memory on disk
- checked `free -h`
- realized WSL was limited to around `15 GiB` RAM and was using swap because the node stack was memory-heavy

**Conclusion**

- swap usage was a symptom of WSL memory pressure

### 3. WSL RAM Too Low

**Concern**

- node services were healthy, but memory pressure remained high

**Resolution**

- created `C:\Users\Anton\.wslconfig`
- assigned:
  - `memory=24GB`
  - `processors=8`
  - `swap=8GB`
- restarted WSL with `wsl --shutdown`

**Conclusion**

- WSL memory increased to around `23 GiB`
- swap dropped to `0B`
- both node services restarted cleanly

### 4. `curl ... | head` Error on Node Exporter

**Concern**

- saw `curl: (23) Failure writing output to destination`

**Resolution**

- understood this was caused by piping into `head`
- confirmed it was not a real exporter problem

**Conclusion**

- Node Exporter was healthy

### 5. Prometheus `/targets` Returned 404 in Terminal

**Concern**

- thought Prometheus might be broken

**Resolution**

- realized `/targets` is not the API endpoint for terminal use
- used `/api/v1/targets` and `/api/v1/query?query=up` instead

**Conclusion**

- Prometheus was healthy and scraping correctly

### 6. Windows Browser Could Not Open Localhost Prometheus

**Concern**

- browser could not connect to `http://localhost:9090/targets` or `http://127.0.0.1:9090/targets`

**Resolution**

- confirmed Prometheus was healthy inside WSL
- used `hostname -I` to obtain the WSL IP
- used the WSL IP when browser access was required

**Conclusion**

- the service was healthy; the access path was the issue

### 7. Grafana Sign-In Looked Like Account Signup

**Concern**

- thought a Grafana account might be required

**Resolution**

- learned that the login page was for the local Grafana server
- used the default local credentials:
  - `admin`
  - `admin`

**Conclusion**

- no cloud account was needed

---

## Recommended First Grafana Queries

### Check Target Health

    up

### Memory Used Percent

    100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))

### Root Disk Free

    node_filesystem_avail_bytes{mountpoint="/"}

### CPU Usage Percent

    100 * (1 - avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))

### Network Receive Rate

    rate(node_network_receive_bytes_total[5m])

---

## Final Result

By the end of this setup, the following were all working:

- **Nethermind** running under `systemd`
- **Lighthouse** running under `systemd`
- **Node Exporter** exposing host metrics on port `9100`
- **Prometheus** scraping metrics on port `9090`
- **Grafana** running on port `3000`
- **Grafana** successfully connected to Prometheus
- PromQL query `up` returning `1` for both `prometheus` and `node`

This established a working local monitoring stack for Ethereum node infrastructure on Ubuntu WSL2.

---

## Recommended File Location in Repo

Suggested path:

    monitoring/prometheus-grafana-setup.md

Alternative:

    nodes/monitoring-stack.md

---

## Short Daily Monitoring Block

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager
    systemctl status prometheus-node-exporter --no-pager
    systemctl status prometheus --no-pager
    systemctl status grafana-server --no-pager
    ss -tulpn | grep -E '9100|9090|3000|8545|8551|30303|9000'
    free -h
    df -h
    journalctl -u nethermind -n 20 --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager

---

## Summary

This setup established a practical monitoring stack on Ubuntu WSL2 using:

- Node Exporter
- Prometheus
- Grafana

alongside a live Ethereum node stack using:

- Nethermind
- Lighthouse

It also documented the real-world troubleshooting process around:

- memory pressure
- swap interpretation
- WSL tuning
- localhost and browser connectivity
- service verification
- Prometheus API validation
- Grafana login and data source configuration

This provides a strong first observability layer for learning and operating blockchain infrastructure.
