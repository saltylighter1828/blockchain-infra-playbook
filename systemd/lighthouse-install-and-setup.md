# Full Ethereum Node Setup on WSL Ubuntu
## Nethermind + Lighthouse + systemd + JWT (Amended and Debugged)

This document records the full setup used to run a **full Ethereum node** on **Ubuntu under WSL2**, using:

- **Nethermind** as the execution client
- **Lighthouse** as the consensus client
- **systemd** for service management
- a shared **JWT secret** for Engine API authentication

It also includes the important **debugging fixes and amendments** discovered during setup.

---

## Purpose

A post-Merge Ethereum node requires **both**:

- an **execution client** such as Nethermind
- a **consensus client** such as Lighthouse

Running Nethermind alone will often produce messages like:

- `Waiting for Forkchoice message from Consensus Layer`
- `Not receiving ForkChoices from the consensus client that are required to sync`

This is expected until a consensus client is installed and connected through the Engine API.

---

## Environment

- Host OS: Windows 11 Pro
- Linux environment: Ubuntu on WSL2
- Service manager: `systemd`
- Execution client: Nethermind
- Consensus client: Lighthouse
- Execution endpoint: `http://127.0.0.1:8551`
- JSON-RPC endpoint: `http://127.0.0.1:8545`
- P2P ports:
  - Nethermind: `30303`
  - Lighthouse: `9000`
- JWT secret path: `/secrets/jwt.hex`

---

## 1. Install required tools

    sudo apt update && sudo apt upgrade -y
    sudo apt install -y curl wget git jq unzip tar openssl

These tools are used for downloads, archive extraction, JSON handling, and generating the JWT secret.

---

## 2. Create the Nethermind service user

    sudo useradd -m -s /bin/bash nethermind

This creates a dedicated Linux user for Nethermind.

---

## 3. Increase file descriptor limits for Nethermind

Create a dedicated limits file:

    sudo bash -c 'echo "nethermind soft nofile 100000" > /etc/security/limits.d/nethermind.conf'
    sudo bash -c 'echo "nethermind hard nofile 100000" >> /etc/security/limits.d/nethermind.conf'

This helps prevent `too many open files` issues for a network-heavy node.

---

## 4. Create the Nethermind environment file

Create the file as the `nethermind` user:

    sudo -u nethermind bash -c 'cat > /home/nethermind/.env <<EOF
    NETHERMIND_CONFIG="mainnet"
    NETHERMIND_HEALTHCHECKSCONFIG_ENABLED="true"
    EOF'

Then secure it:

    sudo chown nethermind:nethermind /home/nethermind/.env
    sudo chmod 600 /home/nethermind/.env

Important amendment:
- the `.env` file should **not** remain world-readable
- final permissions should be:

    -rw------- 1 nethermind nethermind /home/nethermind/.env

---

## 5. Create the JWT secret file

    sudo mkdir -p /secrets
    openssl rand -hex 32 | tr -d "\n" | sudo tee /secrets/jwt.hex
    sudo chmod 600 /secrets/jwt.hex

This file is used for Engine API authentication between Nethermind and Lighthouse.

---

## 6. Configure Nethermind to use the JWT secret

Edit the Nethermind environment file:

    sudo -u nethermind nano /home/nethermind/.env

Add this line:

    NETHERMIND_JSONRPCCONFIG_JWTSECRETFILE="/secrets/jwt.hex"

Then restart Nethermind:

    sudo systemctl restart nethermind
    sudo systemctl status nethermind --no-pager

Important amendment:
- `daemon-reload` is **not required** when only changing `.env`
- it is only needed when changing a `.service` file

---

## 7. Create the Nethermind systemd service

Create the service file:

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
    WantedBy=multi-user.target
    EOF

Then reload and start:

    sudo systemctl daemon-reload
    sudo systemctl enable nethermind
    sudo systemctl start nethermind
    sudo systemctl status nethermind --no-pager

Important amendment:
- `WantedBy=multi-user.target` is preferred over `default.target`

---

## 8. Verify Nethermind before adding Lighthouse

Check process:

    ps aux | grep nethermind

Check ports:

    ss -tulpn | grep -E '8545|8551|30303'

Check RPC:

    curl -X POST http://localhost:8545 \
      -H "Content-Type: application/json" \
      -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

Early on, a result like:

    {"jsonrpc":"2.0","result":"0x0","id":1}

is acceptable and simply means the node is alive but not yet synced.

---

## 9. Download and install Lighthouse

Important amendment:
- the generic URL using `latest/download/lighthouse-x86_64-unknown-linux-gnu.tar.gz` did **not** work
- it downloaded a tiny invalid file instead of a tarball
- the working install used the actual versioned filename

Use:

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

## 10. Create a dedicated Lighthouse service user

    sudo useradd -m -s /bin/bash lighthouse
    sudo mkdir -p /var/lib/lighthouse
    sudo chown -R lighthouse:lighthouse /var/lib/lighthouse

This creates a dedicated Linux user and persistent data directory for Lighthouse.

---

## 11. Fix JWT access for both Nethermind and Lighthouse

This was one of the most important debugging amendments.

### Problem discovered

If `/secrets/jwt.hex` is:

    -rw------- 1 root root /secrets/jwt.hex

then neither `nethermind` nor `lighthouse` can read it.

If it is changed to:

    -rw-r----- 1 root lighthouse /secrets/jwt.hex

then Lighthouse can read it, but Nethermind may lose access.

### Final clean fix

Create a shared group:

    sudo groupadd jwtreaders

Add both service users:

    sudo usermod -aG jwtreaders nethermind
    sudo usermod -aG jwtreaders lighthouse

Set file ownership and permissions:

    sudo chown root:jwtreaders /secrets/jwt.hex
    sudo chmod 640 /secrets/jwt.hex

Verify group membership:

    id nethermind
    id lighthouse

Verify file access:

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_readable || echo nethermind_NOT_readable
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_readable || echo lighthouse_NOT_readable

Final desired state:

- file group = `jwtreaders`
- both service users belong to `jwtreaders`
- both users can read the file
- file is not world-readable

Expected file permissions:

    -rw-r----- 1 root jwtreaders /secrets/jwt.hex

After changing group membership, restart Nethermind so the running process picks up the new group membership:

    sudo systemctl restart nethermind

---

## 12. Manually test Lighthouse before creating the service

Important amendment:
- Lighthouse was manually tested first before converting it to a systemd service
- this made debugging much easier

Run:

    sudo -u lighthouse lighthouse beacon_node \
      --network mainnet \
      --execution-endpoint http://127.0.0.1:8551 \
      --execution-jwt /secrets/jwt.hex \
      --datadir /var/lib/lighthouse \
      --checkpoint-sync-url https://mainnet.checkpoint.sigp.io

Expected good signs:

- Lighthouse starts successfully
- checkpoint sync begins
- no JWT authentication errors
- peers connect
- blocks begin arriving

Typical healthy messages include:

- `Starting checkpoint sync`
- `Loaded checkpoint block and state`
- `Beacon chain initialized`
- `New block received`

Typical expected warnings during early sync include:

- `Head is optimistic`
- `Downloading historical blocks`

These are normal while Nethermind is still syncing.

---

## 13. Stop the manual Lighthouse process before converting to systemd

Important amendment:
- the manual Lighthouse process must be stopped before creating the systemd service
- otherwise:
  - port `9000` may already be in use
  - the datadir may already be locked

Stop it with:

    Ctrl + C

Optional port check:

    ss -tulpn | grep 9000

The port should not be occupied by the manual foreground process when switching to systemd mode.

---

## 14. Create the Lighthouse environment file

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

This file is not a secret, so `644` is appropriate.

Expected permissions:

    -rw-r--r-- 1 root root /etc/lighthouse-beacon.env

---

## 15. Create the Lighthouse systemd service file

Important amendment:
- `network-online.target` is preferred over `network.target`
- `LimitNOFILE=65535` was added as good network-service hygiene

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

Notes:
- `bn` is the short form of `beacon_node`
- both are valid

---

## 16. Start and enable the Lighthouse service

    sudo systemctl daemon-reload
    sudo systemctl enable lighthouse-beacon
    sudo systemctl start lighthouse-beacon
    sudo systemctl status lighthouse-beacon --no-pager

Then check logs:

    journalctl -u lighthouse-beacon -n 50 --no-pager

Healthy output should include:

- `Lighthouse started`
- `Configured network`
- `Data directory initialised`
- `Hot-Cold DB initialized`

---

## 17. Verify both services are healthy

Check service status:

    systemctl status nethermind --no-pager
    systemctl status lighthouse-beacon --no-pager

Check live Lighthouse logs:

    journalctl -u lighthouse-beacon -f

Check live Nethermind logs:

    journalctl -u nethermind -f

---

## 18. Verify the ports

Check ports:

    ss -tulpn | grep -E '8545|8551|30303|9000'

Expected healthy meaning:

- `8545` = Nethermind JSON-RPC, bound to `127.0.0.1`
- `8551` = Nethermind Engine API, bound locally
- `30303` = Nethermind P2P port
- `9000` = Lighthouse P2P port

Desired safety posture:
- RPC stays local
- P2P ports are exposed for peer connectivity

---

## 19. What changed after Lighthouse was connected

Before Lighthouse:
- Nethermind showed:
  - `Waiting for Forkchoice message from Consensus Layer`

After Lighthouse:
- Nethermind began receiving:
  - `Received ForkChoice`
  - `Received New Block`
  - `Syncing... Inserting block`

This confirmed the execution and consensus layers were correctly connected.

---

## 20. Expected normal warnings during sync

These warnings are normal during initial sync and do not necessarily indicate failure:

### In Lighthouse
- `Head is optimistic`
- `Downloading historical blocks`
- occasional `Backfill sync failed`
- backfill pausing due to insufficient synced peers

Meaning:
- Lighthouse has consensus head data
- Nethermind is still syncing execution data
- historical backfill continues in the background

### In Nethermind
- memory usage rising during sync
- snap sync phase progress messages
- fast sync / snap sync state transitions

Meaning:
- execution sync is progressing normally

---

## 21. Useful daily commands

### Nethermind

    sudo systemctl start nethermind
    sudo systemctl stop nethermind
    sudo systemctl restart nethermind
    sudo systemctl status nethermind --no-pager
    journalctl -u nethermind -f
    journalctl -u nethermind -n 50 --no-pager

### Lighthouse

    sudo systemctl start lighthouse-beacon
    sudo systemctl stop lighthouse-beacon
    sudo systemctl restart lighthouse-beacon
    sudo systemctl status lighthouse-beacon --no-pager
    journalctl -u lighthouse-beacon -f
    journalctl -u lighthouse-beacon -n 50 --no-pager

### Full status

    ss -tulpn | grep -E '8545|8551|30303|9000'
    ps aux | grep nethermind
    ps aux | grep lighthouse
    df -h
    du -sh /home/nethermind/data
    du -sh /var/lib/lighthouse

---

## 22. Final architecture

The finished setup looks like this:

- **Nethermind**
  - execution client
  - runs under user `nethermind`
  - systemd-managed
  - exposes:
    - `8545` JSON-RPC locally
    - `8551` Engine API locally
    - `30303` P2P

- **Lighthouse**
  - consensus client
  - runs under user `lighthouse`
  - systemd-managed
  - exposes:
    - `9000` P2P

- **JWT secret**
  - stored at `/secrets/jwt.hex`
  - owned by `root:jwtreaders`
  - permissions `640`
  - readable by both service users through shared group membership

---

## 23. Outcome

At the end of this setup, the node achieved:

- Nethermind running correctly as the execution client
- Lighthouse running correctly as the consensus client
- both clients connected through the Engine API
- JWT authentication working securely
- both services managed by systemd
- auto-start on boot enabled
- proper Linux user separation
- correct port exposure
- a fully functioning post-Merge Ethereum node architecture

---

## 24. What I learned

- how execution and consensus clients fit together after the Merge
- why Nethermind alone waits for forkchoice
- how to create and secure a JWT secret
- how to share a JWT safely between two service users
- how to use `.env` files with systemd services
- how to test a client manually before automating it
- how to convert a working manual process into a systemd service
- how to inspect services with `systemctl`
- how to inspect logs with `journalctl`
- how to verify ports with `ss`
- how to reason about sync state from logs

---

## 25. Final note

This setup was not just a straight-line install. It required several real-world debugging amendments:

- fixing `.env` permissions
- fixing JWT file readability
- avoiding breaking Nethermind while making Lighthouse readable
- creating a shared group for JWT access
- using the correct Lighthouse release tarball
- manually testing Lighthouse before automating it
- stopping the foreground process before enabling the systemd service
- improving the service unit with `network-online.target` and `LimitNOFILE`

That debugging process was part of the learning and is exactly what makes this setup valuable.
