# Nethermind Install and Service Setup on Ubuntu WSL2

This document records the installation, configuration, and validation of Nethermind as the Ethereum execution client on Ubuntu under WSL2, including service management with `systemd`, runtime validation, and the initial execution-only node state prior to consensus client integration. :contentReference[oaicite:0]{index=0}

---

## Purpose

The goal of this setup was to install Nethermind cleanly, run it under `systemd`, and verify that the execution layer was operating correctly on Ubuntu under WSL2.

This setup established:

- Nethermind installed from the official Ubuntu package source
- a dedicated Linux service user
- a persistent data directory
- an environment file for runtime configuration
- a `systemd` unit for controlled service management
- validation through logs, ports, process inspection, and disk growth

At this stage, Nethermind was functioning as an **execution client** only. Post-Merge, that means it could run correctly on its own, but still required a consensus client to form a complete Ethereum node.

---

## Environment

- Host OS: Windows 11 Pro
- Linux environment: Ubuntu on WSL2
- Service manager: `systemd`
- Execution client: Nethermind
- Initial data directory: `/home/nethermind/data`

---

## 1. Update Ubuntu

Refresh package lists and install available updates before adding new software.

    sudo apt update && sudo apt upgrade -y

---

## 2. Install Base Tools

Install the common tools used for downloads, package management, process inspection, networking, and troubleshooting.

    sudo apt install -y curl wget git jq unzip htop net-tools lsof build-essential software-properties-common ca-certificates

These tools are useful throughout the rest of the node setup and debugging workflow.

---

## 3. Confirm `systemd` Is Running

Verify that `systemd` is running as PID 1.

    ps -p 1 -o comm=

Expected output:

    systemd

This confirms that the WSL environment is using `systemd`, which is required for proper service management.

---

## 4. Add the Nethermind PPA and Install Nethermind

Install Nethermind from the official Ubuntu package source.

    sudo add-apt-repository ppa:nethermindeth/nethermind -y
    sudo apt update
    sudo apt install -y nethermind

This installs the Nethermind binary into the system path.

---

## 5. Verify the Installation

Confirm that the Nethermind binary is installed and available.

    which nethermind
    nethermind --version
    nethermind --help | head -n 20

This verifies:

- the install path
- the installed version
- that the binary is runnable

---

## 6. Create a Dedicated Service User

Create a dedicated Linux user for the Nethermind service and increase open-file limits.

    sudo useradd -m -s /bin/bash nethermind
    sudo bash -c 'echo "nethermind soft nofile 100000" > /etc/security/limits.d/nethermind.conf'
    sudo bash -c 'echo "nethermind hard nofile 100000" >> /etc/security/limits.d/nethermind.conf'

This keeps the service isolated from the main user account and improves readiness for a network-heavy workload.

---

## 7. Create the Data Directory and Environment File

Create the Nethermind data directory and the basic environment configuration file.

    sudo -u nethermind mkdir -p /home/nethermind/data

    sudo -u nethermind bash -c 'cat > /home/nethermind/.env <<EOF
    NETHERMIND_CONFIG="mainnet"
    NETHERMIND_HEALTHCHECKSCONFIG_ENABLED="true"
    EOF'

This sets up:

- a dedicated data directory
- a small environment file for service configuration

### Notes
- `NETHERMIND_CONFIG="mainnet"` selects Ethereum mainnet
- `NETHERMIND_HEALTHCHECKSCONFIG_ENABLED="true"` enables health-check functionality

---

## 8. Create the `systemd` Service File

Create the service unit used to run Nethermind under `systemd`.

    sudo tee /etc/systemd/system/nethermind.service > /dev/null <<'EOF'
    [Unit]
    Description=Nethermind node
    Documentation=https://docs.nethermind.io
    After=network.target

    [Service]
    User=nethermind
    Group=nethermind
    EnvironmentFile=/home/nethermind/.env
    WorkingDirectory=/home/nethermind
    ExecStart=/usr/bin/nethermind --data-dir /home/nethermind/data
    Restart=on-failure
    LimitNOFILE=1000000

    [Install]
    WantedBy=default.target
    EOF

This service configuration:

- runs Nethermind as the `nethermind` user
- loads the environment file automatically
- uses `/home/nethermind/data` as the data directory
- restarts on failure
- increases the open-file limit for a networked service

---

## 9. Reload `systemd` and Start the Service

After creating the service file, reload `systemd`, start the service, and enable it to start automatically.

    sudo systemctl daemon-reload
    sudo systemctl start nethermind
    sudo systemctl enable nethermind

### What each command does
- `daemon-reload` makes `systemd` re-read unit files
- `start` launches the service immediately
- `enable` configures automatic startup

---

## 10. Check Service Status

Confirm that the service is loaded and running.

    sudo systemctl status nethermind --no-pager

A healthy result should show:

- `Loaded: loaded`
- `Active: active (running)`

This is the main confirmation that the service is running correctly under `systemd`.

---

## 11. View Logs

Use `journalctl` to inspect Nethermind logs.

### Follow live logs

    journalctl -u nethermind -f

### View recent logs

    journalctl -u nethermind -n 50 --no-pager

These logs are useful for validating startup behaviour, sync progress, Engine API readiness, and any errors or warnings.

---

## 12. Check Process, Ports, and Disk Usage

### Process Check

    ps aux | grep nethermind

Use this to confirm that the Nethermind process is present and running.

### Port Check

    ss -tulpn | grep -E '8545|8551|30303'

Use this to confirm that key ports are listening.

### Disk Usage

    df -h
    sudo du -sh /home/nethermind/data

Use these to verify:

- available filesystem space
- Nethermind data directory growth over time

These checks help confirm that the service is not only running, but writing data and exposing the expected interfaces.

---

## 13. Port Notes

The key ports in this execution-client setup are:

- `30303` → Ethereum execution-layer peer-to-peer networking
- `8545` → JSON-RPC
- `8551` → Engine API for execution ↔ consensus communication

### Important Context

At this stage, Nethermind was running in execution-only mode.

That means:

- JSON-RPC could still function
- peer-to-peer networking could still function
- logs could still show healthy service behaviour

But Nethermind would still wait for a consensus client to provide forkchoice updates, which is expected until a consensus client is added.

---

## 14. Useful Service Commands

These commands are useful for routine operation and recovery.

    sudo systemctl start nethermind
    sudo systemctl stop nethermind
    sudo systemctl restart nethermind
    sudo systemctl status nethermind --no-pager
    journalctl -u nethermind -f
    journalctl -u nethermind -n 50 --no-pager

---

## 15. Validation Summary

The setup was validated at four layers:

### Binary Layer
- Nethermind installed successfully
- binary available in the system path
- version and help output working

### Service Layer
- `systemd` unit created successfully
- service starts and runs under the dedicated `nethermind` user
- service can be managed with `systemctl`

### Network Layer
- expected ports listening
- JSON-RPC and Engine API endpoints exposed
- peer-to-peer networking active

### Storage Layer
- data directory created successfully
- chain data begins to accumulate over time
- disk space can be monitored with `df` and `du`

---

## 16. Screenshots

### Service Status
![Nethermind service status](../assets/screenshots/nethermind/systemctl-status.png)

### Recent Logs
![Nethermind recent logs](../assets/screenshots/nethermind/journalctl-logs.png)

### Port Check
![Nethermind ports](../assets/screenshots/nethermind/ports-check.png)

### Disk and Data Usage
![Nethermind disk and data usage](../assets/screenshots/nethermind/disk-and-data-usage.png)

---

## 17. What I Learned

This setup established several important operational ideas:

- how to install Nethermind on Ubuntu with `apt`
- how to verify a binary using `--version` and `--help`
- how to create a dedicated Linux service user
- how to create and manage a `systemd` unit
- how to inspect service health with `systemctl` and `journalctl`
- how to validate open ports with `ss`
- how to inspect storage growth with `du`
- why an execution client alone still waits for consensus-layer forkchoice updates after the Merge

---

## 18. Outcome

At the end of this setup:

- Nethermind was installed successfully
- the service was running under `systemd`
- ports were listening as expected
- the data directory was being created and written to
- the environment was ready for the next major layers:
  - consensus client integration
  - monitoring and observability
  - operational scripting and automation

This completed the execution-client foundation for the broader Ethereum node stack.
