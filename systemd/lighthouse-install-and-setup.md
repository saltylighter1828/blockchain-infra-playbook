# Lighthouse Install and Setup on WSL Ubuntu
## Consensus Client Setup for Nethermind

This document records the commands used to install Lighthouse as the Ethereum consensus client on Ubuntu running under WSL2, connect it to Nethermind through the Engine API, debug the setup, and finally run Lighthouse as a `systemd` service.

---

## Purpose

My current setup already includes:

- **Nethermind** as the execution client
- Ubuntu on WSL2
- `systemd` for service management

Nethermind is running correctly, but execution-only mode is not enough for a full post-Merge Ethereum node. Lighthouse provides the **consensus layer** and communicates with Nethermind through the **Engine API** using a shared JWT secret.

---

## Environment

- Host OS: Windows 11
- Linux environment: Ubuntu on WSL2
- Service manager: `systemd`
- Execution client: Nethermind
- Consensus client: Lighthouse
- Execution endpoint: `http://127.0.0.1:8551`
- Shared JWT secret path: `/secrets/jwt.hex`

---

## 1. Install required tools

    sudo apt update && sudo apt upgrade -y
    sudo apt install -y curl wget git jq unzip tar openssl

These tools are used for downloads, archive extraction, and verification.

---

## 2. Download and install Lighthouse

### Important amendment

The generic URL using `latest/download/lighthouse-x86_64-unknown-linux-gnu.tar.gz` did **not** work. It downloaded an invalid tiny file instead of the real archive.

The working install used the versioned release tarball.

    cd /tmp
    curl -LO https://github.com/sigp/lighthouse/releases/download/v8.1.3/lighthouse-v8.1.3-x86_64-unknown-linux-gnu.tar.gz
    tar -xzf lighthouse-v8.1.3-x86_64-unknown-linux-gnu.tar.gz
    sudo install -m 0755 lighthouse /usr/local/bin/lighthouse

Verify installation:

    which lighthouse
    lighthouse --version

Expected output includes something like:

    Lighthouse v8.1.3-176cce5

---

## 3. Create a dedicated Lighthouse user and data directory

    sudo useradd -m -s /bin/bash lighthouse
    sudo mkdir -p /var/lib/lighthouse
    sudo chown -R lighthouse:lighthouse /var/lib/lighthouse

This creates a dedicated Linux user and persistent data directory for Lighthouse.

---

## 4. Ensure Lighthouse can read the shared JWT secret

Lighthouse and Nethermind both need to read the same JWT secret file:

    /secrets/jwt.hex

### Problem discovered

If the file is owned like this:

    -rw------- 1 root root /secrets/jwt.hex

then Lighthouse cannot read it.

### Initial check

    sudo -u lighthouse test -r /secrets/jwt.hex && echo readable || echo NOT_readable

### Clean final fix

A shared group was created so both `nethermind` and `lighthouse` can read the file without making it public.

    sudo groupadd jwtreaders
    sudo usermod -aG jwtreaders nethermind
    sudo usermod -aG jwtreaders lighthouse
    sudo chown root:jwtreaders /secrets/jwt.hex
    sudo chmod 640 /secrets/jwt.hex

Verify:

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_readable || echo nethermind_NOT_readable
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_readable || echo lighthouse_NOT_readable

Expected final state:

- both users can read the file
- file is not world-readable
- file permissions look like:

    -rw-r----- 1 root jwtreaders /secrets/jwt.hex

---

## 5. Manually test Lighthouse before creating the service

### Important amendment

Lighthouse was tested manually first before converting it into a `systemd` service. This made debugging much easier.

Run:

    sudo -u lighthouse lighthouse beacon_node \
      --network mainnet \
      --execution-endpoint http://127.0.0.1:8551 \
      --execution-jwt /secrets/jwt.hex \
      --datadir /var/lib/lighthouse \
      --checkpoint-sync-url https://mainnet.checkpoint.sigp.io

### What this does

- `beacon_node` starts the consensus client
- `--execution-endpoint` points to Nethermind’s Engine API
- `--execution-jwt` uses the shared JWT secret
- `--datadir` stores Lighthouse data in a dedicated directory
- `--checkpoint-sync-url` speeds up sync significantly

### Good signs observed

Healthy output included:

- `Lighthouse started`
- `Configured network`
- `Starting checkpoint sync`
- `Loaded checkpoint block and state`
- `Beacon chain initialized`
- `New block received`

### Expected warnings during early sync

These warnings were seen and are normal during setup:

- `Head is optimistic`
- `Downloading historical blocks`
- occasional backfill sync pauses or retries

This is expected while Nethermind is still syncing the execution layer.

---

## 6. Stop the manual Lighthouse process before switching to systemd

### Important amendment

Before creating the service, the manually running Lighthouse process must be stopped.

Stop it with:

    Ctrl + C

This avoids:
- port conflicts on `9000`
- datadir lock issues

Optional check:

    ss -tulpn | grep 9000

---

## 7. Create the Lighthouse environment file

    sudo tee /etc/lighthouse-beacon.env > /dev/null <<'EOF'
    LIGHTHOUSE_DATADIR="/var/lib/lighthouse"
    LIGHTHOUSE_NETWORK="mainnet"
    LIGHTHOUSE_EXECUTION_ENDPOINT="http://127.0.0.1:8551"
    LIGHTHOUSE_JWT_SECRET="/secrets/jwt.hex"
    LIGHTHOUSE_CHECKPOINT_SYNC_URL="https://mainnet.checkpoint.sigp.io"
    EOF

Verify contents:

    cat /etc/lighthouse-beacon.env

Set permissions:

    sudo chmod 644 /etc/lighthouse-beacon.env
    ls -l /etc/lighthouse-beacon.env

Expected permissions:

    -rw-r--r-- 1 root root /etc/lighthouse-beacon.env

This file is configuration only, not a secret.

---

## 8. Create the Lighthouse systemd service file

### Important amendments

The final service used:
- `network-online.target` instead of just `network.target`
- `LimitNOFILE=65535` for network-heavy service hygiene
- `lighthouse bn`, which is the short form of `lighthouse beacon_node`

Create the service file:

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

---

## 9. Reload systemd and start Lighthouse

    sudo systemctl daemon-reload
    sudo systemctl enable lighthouse-beacon
    sudo systemctl start lighthouse-beacon
    sudo systemctl status lighthouse-beacon --no-pager

Check logs:

    journalctl -u lighthouse-beacon -n 50 --no-pager

Healthy output included:

- `Lighthouse started`
- `Configured network`
- `Data directory initialised`
- `Hot-Cold DB initialized`
- `Blob DB initialized`

---

## 10. Verify Lighthouse is healthy

Check status:

    systemctl status lighthouse-beacon --no-pager

Follow logs live:

    journalctl -u lighthouse-beacon -f

Check relevant port:

    ss -tulpn | grep -E '9000'

Expected:
- UDP and TCP listeners on port `9000`
- active service status
- ongoing sync or block messages

---

## 11. Confirm Lighthouse is talking to Nethermind

After Lighthouse was connected, Nethermind began receiving:

- `Received ForkChoice`
- `Received New Block`
- `Syncing... Inserting block`

This confirmed that Lighthouse and Nethermind were correctly connected through the Engine API.

Lighthouse also showed:
- `Head is optimistic`
- `Synced`
- `New block received`

This was expected while Nethermind was still syncing and had not fully verified the execution layer yet.

---

## 12. Expected normal warnings during sync

These are normal during early sync:

### In Lighthouse

- `Head is optimistic`
- `Downloading historical blocks`
- backfill sync pauses or retries
- `chain not fully verified, block and attestation production disabled until execution engine syncs`

Meaning:
- consensus is connected
- execution is still catching up
- node is still healthy

---

## 13. Useful daily Lighthouse commands

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

At the end of this setup, Lighthouse achieved:

- installed successfully from a release tarball
- running under a dedicated `lighthouse` user
- storing data in `/var/lib/lighthouse`
- reading the shared JWT secret securely
- connecting to Nethermind through the Engine API
- completing checkpoint sync
- receiving live beacon blocks
- running as a `systemd` service
- auto-starting on boot

---

## 15. What I learned

- how to install Lighthouse from a release binary
- how to connect Lighthouse to Nethermind through `8551`
- why a shared JWT secret is required
- how to safely share that JWT between multiple service users
- why manual testing before automation is important
- how to convert Lighthouse into a `systemd` service
- how to inspect Lighthouse with `systemctl`, `journalctl`, and `ss`
- what `Head is optimistic` means during sync

---

## 16. Final note

This setup was not just a straight install. It required several real debugging amendments:

- using the correct versioned Lighthouse tarball
- fixing JWT readability for Lighthouse
- avoiding breaking Nethermind while fixing Lighthouse access
- creating a shared group for JWT access
- manually testing Lighthouse before service creation
- stopping the foreground process before enabling the systemd service
- improving the service unit with `network-online.target` and `LimitNOFILE`

That debugging process was part of the real learning.
