# Lighthouse Install and Setup on Ubuntu WSL2
## Consensus Client Integration for Nethermind

This document records the installation, configuration, and validation of Lighthouse as the Ethereum consensus client on Ubuntu under WSL2, paired with Nethermind as the execution client.

It covers the full setup path, including:

- installing Lighthouse from a release binary
- connecting Lighthouse to Nethermind through the Engine API
- securing execution-consensus communication with a shared JWT secret
- validating the setup manually before automation
- converting Lighthouse into a `systemd` service
- debugging real issues encountered during integration

The result is a working post-Merge Ethereum node stack with separate execution and consensus clients running under `systemd`.

---

## Purpose

The environment already includes:

- **Nethermind** as the execution client
- Ubuntu on WSL2
- `systemd` for service management

Nethermind was operating correctly, but an execution client alone is not enough for a complete post-Merge Ethereum node. Lighthouse provides the **consensus layer** and communicates with Nethermind over the **Engine API** using a shared JWT secret.

This setup completes the node architecture by adding the consensus side of the stack.

---

## Environment

- Host OS: Windows 11
- Linux environment: Ubuntu on WSL2
- Service manager: `systemd`
- Execution client: Nethermind
- Consensus client: Lighthouse
- Execution endpoint: `http://127.0.0.1:8551`
- Shared JWT secret: `/secrets/jwt.hex`

---

## 1. Install Required Tools

These packages are used for downloads, extraction, and validation.

    sudo apt update && sudo apt upgrade -y
    sudo apt install -y curl wget git jq unzip tar openssl

---

## 2. Download and Install Lighthouse

### Important Installation Note

The generic GitHub release URL using `latest/download/lighthouse-x86_64-unknown-linux-gnu.tar.gz` did **not** work correctly. It downloaded an invalid tiny file instead of the real archive.

The working installation used the versioned release tarball.

    cd /tmp
    curl -LO https://github.com/sigp/lighthouse/releases/download/v8.1.3/lighthouse-v8.1.3-x86_64-unknown-linux-gnu.tar.gz
    tar -xzf lighthouse-v8.1.3-x86_64-unknown-linux-gnu.tar.gz
    sudo install -m 0755 lighthouse /usr/local/bin/lighthouse

### Verify Installation

    which lighthouse
    lighthouse --version

Expected output includes something like:

    Lighthouse v8.1.3-176cce5

This confirms the binary is installed and available on the system path.

---

## 3. Create a Dedicated Lighthouse User and Data Directory

A dedicated service user and persistent data directory keep the setup cleaner and more operationally consistent.

    sudo useradd -m -s /bin/bash lighthouse
    sudo mkdir -p /var/lib/lighthouse
    sudo chown -R lighthouse:lighthouse /var/lib/lighthouse

This creates:

- a dedicated Linux user for Lighthouse
- a persistent data directory at `/var/lib/lighthouse`

---

## 4. Ensure Lighthouse Can Read the Shared JWT Secret

Both Lighthouse and Nethermind need read access to the same JWT secret:

    /secrets/jwt.hex

### Problem Identified

If the file is owned like this:

    -rw------- 1 root root /secrets/jwt.hex

then Lighthouse cannot read it.

### Initial Readability Check

    sudo -u lighthouse test -r /secrets/jwt.hex && echo readable || echo NOT_readable

### Clean Final Fix

A shared group was created so both `nethermind` and `lighthouse` could read the file without making it public.

    sudo groupadd jwtreaders
    sudo usermod -aG jwtreaders nethermind
    sudo usermod -aG jwtreaders lighthouse
    sudo chown root:jwtreaders /secrets/jwt.hex
    sudo chmod 640 /secrets/jwt.hex

### Verify Access

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_readable || echo nethermind_NOT_readable
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_readable || echo lighthouse_NOT_readable

Expected final state:

- both users can read the file
- the file is not world-readable
- permissions look like:

    -rw-r----- 1 root jwtreaders /secrets/jwt.hex

This is the cleanest final arrangement because it preserves security while allowing the two services to share the same secret.

---

## 5. Manually Test Lighthouse Before Creating the Service

### Why Manual Testing Matters

Lighthouse was tested manually before creating the `systemd` service. This made debugging easier because startup messages and failures were visible immediately in the terminal.

### Manual Launch Command

    sudo -u lighthouse lighthouse beacon_node \
      --network mainnet \
      --execution-endpoint http://127.0.0.1:8551 \
      --execution-jwt /secrets/jwt.hex \
      --datadir /var/lib/lighthouse \
      --checkpoint-sync-url https://mainnet.checkpoint.sigp.io

### What This Does

- starts the beacon node
- points Lighthouse at Nethermind’s Engine API
- uses the shared JWT secret for authentication
- stores Lighthouse data in `/var/lib/lighthouse`
- uses checkpoint sync to accelerate initial sync

### Healthy Signs Observed

Expected healthy output included:

- `Lighthouse started`
- `Configured network`
- `Starting checkpoint sync`
- `Loaded checkpoint block and state`
- `Beacon chain initialized`
- `New block received`

### Expected Warnings During Early Sync

Warnings seen during early sync included:

- `Head is optimistic`
- `Downloading historical blocks`
- occasional backfill sync pauses or retries

These were normal while Nethermind was still syncing the execution layer.

---

## 6. Stop the Manual Lighthouse Process Before Switching to systemd

Before creating the service unit, the manually running Lighthouse process must be stopped.

Stop it with:

    Ctrl + C

This avoids:

- port conflicts on `9000`
- datadir lock issues
- two Lighthouse processes competing for the same resources

### Optional Port Check

    ss -tulpn | grep 9000

---

## 7. Create the Lighthouse Environment File

Using an environment file keeps the service definition cleaner and makes configuration easier to review.

    sudo tee /etc/lighthouse-beacon.env > /dev/null <<'EOF'
    LIGHTHOUSE_DATADIR="/var/lib/lighthouse"
    LIGHTHOUSE_NETWORK="mainnet"
    LIGHTHOUSE_EXECUTION_ENDPOINT="http://127.0.0.1:8551"
    LIGHTHOUSE_JWT_SECRET="/secrets/jwt.hex"
    LIGHTHOUSE_CHECKPOINT_SYNC_URL="https://mainnet.checkpoint.sigp.io"
    EOF

### Verify Contents

    cat /etc/lighthouse-beacon.env

### Set Permissions

    sudo chmod 644 /etc/lighthouse-beacon.env
    ls -l /etc/lighthouse-beacon.env

Expected permissions:

    -rw-r--r-- 1 root root /etc/lighthouse-beacon.env

This file contains configuration only, not secrets, so standard readable permissions are acceptable.

---

## 8. Create the Lighthouse systemd Service File

### Final Service Design Notes

The final service unit used:

- `network-online.target` instead of only `network.target`
- `LimitNOFILE=65535` for better network-service hygiene
- `lighthouse bn`, which is the short form of `lighthouse beacon_node`

### Create the Service File

    sudo tee /etc/systemd/system/lighthouse-beacon.service > /dev/null <<'EOF'
    [Unit]
    Description=Lighthouse Beacon Node
    Documentation=https://lighthouse-book.sigmaprime.io/
    After=network-online.target nethermind.service
    Wants=network-online.target nethermind.service

    [Service]
    User=lighthouse
    Group=lighthouse
    EnvironmentFile=/etc/lighthouse-beacon.env
    Type=simple
    Restart=on-failure
    RestartSec=5
    LimitNOFILE=65535
    ExecStart=/usr/local/bin/lighthouse bn \
        --network ${LIGHTHOUSE_NETWORK} \
        --datadir ${LIGHTHOUSE_DATADIR} \
        --execution-endpoint ${LIGHTHOUSE_EXECUTION_ENDPOINT} \
        --execution-jwt ${LIGHTHOUSE_JWT_SECRET} \
        --checkpoint-sync-url ${LIGHTHOUSE_CHECKPOINT_SYNC_URL}

    [Install]
    WantedBy=multi-user.target
    EOF

This service definition keeps the setup simple while still covering the core runtime requirements for Lighthouse in this environment.

---

## 9. Reload systemd and Start Lighthouse

Once the unit file is in place, reload `systemd`, enable the service, and start it.

    sudo systemctl daemon-reload
    sudo systemctl enable lighthouse-beacon
    sudo systemctl start lighthouse-beacon
    sudo systemctl status lighthouse-beacon --no-pager

### Check Logs

    journalctl -u lighthouse-beacon -n 50 --no-pager

Healthy output included:

- `Lighthouse started`
- `Configured network`
- `Data directory initialised`
- `Hot-Cold DB initialized`
- `Blob DB initialized`

---

## 10. Verify Lighthouse Is Healthy

### Check Service Status

    systemctl status lighthouse-beacon --no-pager

### Follow Logs Live

    journalctl -u lighthouse-beacon -f

### Check P2P Port

    ss -tulpn | grep -E '9000'

Expected healthy state:

- TCP and UDP listeners on port `9000`
- service status showing `active (running)`
- logs continuing to show sync or block activity

---

## 11. Confirm Lighthouse Is Talking to Nethermind

Once Lighthouse was connected successfully, Nethermind began receiving logs such as:

- `Received ForkChoice`
- `Received New Block`
- `Syncing... Inserting block`

This confirmed that Lighthouse and Nethermind were communicating correctly through the Engine API.

Lighthouse also showed messages such as:

- `Head is optimistic`
- `Synced`
- `New block received`

This was expected while Nethermind was still catching up on the execution side.

---

## 12. Expected Normal Warnings During Sync

Some warnings are expected during early sync and do not necessarily indicate a broken setup.

### In Lighthouse

- `Head is optimistic`
- `Downloading historical blocks`
- backfill sync pauses or retries
- `chain not fully verified, block and attestation production disabled until execution engine syncs`

### What These Mean

These messages generally mean:

- consensus is connected correctly
- execution is still catching up
- the node is healthy, but not yet fully settled

In other words, these warnings are often part of a normal early-sync state rather than a sign of failure.

---

## 13. Useful Daily Lighthouse Commands

These commands are useful for routine operations and validation.

    sudo systemctl start lighthouse-beacon
    sudo systemctl stop lighthouse-beacon
    sudo systemctl restart lighthouse-beacon
    sudo systemctl status lighthouse-beacon --no-pager
    journalctl -u lighthouse-beacon -f
    journalctl -u lighthouse-beacon -n 50 --no-pager
    ss -tulpn | grep -E '9000'
    ps aux | grep lighthouse
    du -sh /var/lib/lighthouse

---

## 14. Outcome

At the end of this setup, Lighthouse was:

- installed successfully from a release tarball
- running under a dedicated `lighthouse` user
- storing data in `/var/lib/lighthouse`
- reading the shared JWT secret securely
- connecting to Nethermind through the Engine API
- completing checkpoint sync
- receiving live beacon blocks
- running as a `systemd` service
- configured to auto-start with the system

This completed the consensus side of the node stack.

---

## 15. What I Learned

This setup clarified several important operational concepts:

- how to install Lighthouse from a release binary
- how to connect Lighthouse to Nethermind through port `8551`
- why a shared JWT secret is required
- how to safely share that JWT between multiple service users
- why manual testing before automation is valuable
- how to convert Lighthouse into a `systemd` service
- how to inspect Lighthouse with `systemctl`, `journalctl`, and `ss`
- what `Head is optimistic` means during sync

---

## 16. Final Note

This setup was not just a straight install. It required several real debugging amendments along the way, including:

- using the correct versioned Lighthouse tarball
- fixing JWT readability for Lighthouse
- avoiding breakage to Nethermind while adjusting JWT access
- creating a shared group for JWT access
- manually testing Lighthouse before service creation
- stopping the foreground process before enabling the `systemd` service
- improving the service unit with `network-online.target` and `LimitNOFILE`

That debugging process was part of the real value of the setup. It turned the install into a more complete infrastructure learning exercise rather than a one-command deployment.
