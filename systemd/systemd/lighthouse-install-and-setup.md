# Lighthouse Install and Setup on WSL Ubuntu

This document records the commands used to install Lighthouse as the Ethereum consensus client on Ubuntu running under WSL2, configure it as a `systemd` service, and connect it to Nethermind through the Engine API.

---

## Purpose

My current setup already includes:

- **Nethermind** as the execution client
- Ubuntu on WSL2
- `systemd` for service management

Nethermind is running correctly, but its logs show messages such as:

- `Waiting for Forkchoice message from Consensus Layer`
- `Not receiving ForkChoices from the consensus client that are required to sync`

These messages are expected when an execution client is running without a consensus client. Lighthouse will provide the consensus layer connection and communicate with Nethermind through the Engine API.

---

## Environment

- Host OS: Windows 11 Pro
- Linux environment: Ubuntu on WSL2
- Service manager: `systemd`
- Execution client: Nethermind
- Consensus client: Lighthouse
- Execution endpoint: `http://127.0.0.1:8551`
- JWT secret path: `/secrets/jwt.hex`

---

## 1. Install required tools

    sudo apt update && sudo apt upgrade -y
    sudo apt install -y curl wget git jq unzip tar openssl

These tools are used for downloads, archive extraction, and creating the JWT secret.

---

## 2. Create the JWT secret file

    sudo mkdir -p /secrets
    openssl rand -hex 32 | tr -d "\n" | sudo tee /secrets/jwt.hex
    sudo chmod 600 /secrets/jwt.hex

This file will be used by Lighthouse and Nethermind to authenticate Engine API communication.

---

## 3. Configure Nethermind to use the same JWT secret

Edit the Nethermind environment file:

    sudo nano /home/nethermind/.env

Add this line if it is not already present:

    NETHERMIND_JSONRPCCONFIG_JWTSECRETFILE="/secrets/jwt.hex"

Then reload and restart Nethermind:

    sudo systemctl daemon-reload
    sudo systemctl restart nethermind
    sudo systemctl status nethermind --no-pager

---

## 4. Download Lighthouse

Go to the official Lighthouse releases page and download the latest Linux x86_64 release.

Replace `<VERSION>` below with the release version you choose.

    cd /tmp
    curl -LO https://github.com/sigp/lighthouse/releases/download/<VERSION>/lighthouse-<VERSION>-x86_64-unknown-linux-gnu.tar.gz
    tar -xzf lighthouse-<VERSION>-x86_64-unknown-linux-gnu.tar.gz
    sudo install -m 0755 lighthouse /usr/local/bin/lighthouse

Verify installation:

    which lighthouse
    lighthouse --version

---

## 5. Create a dedicated Lighthouse service user

    sudo useradd -m -s /bin/bash lighthouse
    sudo mkdir -p /var/lib/lighthouse
    sudo chown -R lighthouse:lighthouse /var/lib/lighthouse

This creates a dedicated Linux user and data directory for the Lighthouse beacon node.

---

## 6. Create the Lighthouse environment file

    sudo tee /etc/lighthouse-beacon.env > /dev/null <<'EOF'
    LIGHTHOUSE_DATADIR="/var/lib/lighthouse"
    LIGHTHOUSE_NETWORK="mainnet"
    LIGHTHOUSE_EXECUTION_ENDPOINT="http://127.0.0.1:8551"
    LIGHTHOUSE_JWT_SECRET="/secrets/jwt.hex"
    LIGHTHOUSE_CHECKPOINT_SYNC_URL="https://mainnet.checkpoint.sigp.io"
    EOF

Notes:
- `LIGHTHOUSE_EXECUTION_ENDPOINT` points to Nethermind’s Engine API
- `LIGHTHOUSE_JWT_SECRET` points to the same JWT file used by Nethermind
- `LIGHTHOUSE_CHECKPOINT_SYNC_URL` can speed up beacon node sync

---

## 7. Create the Lighthouse systemd service file

    sudo tee /etc/systemd/system/lighthouse-beacon.service > /dev/null <<'EOF'
    [Unit]
    Description=Lighthouse Beacon Node
    Documentation=https://lighthouse-book.sigmaprime.io/
    After=network.target nethermind.service
    Wants=nethermind.service

    [Service]
    User=lighthouse
    Group=lighthouse
    EnvironmentFile=/etc/lighthouse-beacon.env
    Type=simple
    Restart=on-failure
    RestartSec=5
    ExecStart=/usr/local/bin/lighthouse bn \
        --network ${LIGHTHOUSE_NETWORK} \
        --datadir ${LIGHTHOUSE_DATADIR} \
        --execution-endpoint ${LIGHTHOUSE_EXECUTION_ENDPOINT} \
        --execution-jwt ${LIGHTHOUSE_JWT_SECRET} \
        --checkpoint-sync-url ${LIGHTHOUSE_CHECKPOINT_SYNC_URL}

    [Install]
    WantedBy=multi-user.target
    EOF

This creates the `systemd` unit for the Lighthouse beacon node.

---

## 8. Reload systemd and start Lighthouse

    sudo systemctl daemon-reload
    sudo systemctl start lighthouse-beacon
    sudo systemctl enable lighthouse-beacon

Then verify status:

    sudo systemctl status lighthouse-beacon --no-pager

---

## 9. Check Lighthouse logs

Follow live logs:

    journalctl -u lighthouse-beacon -f

View recent logs without live follow:

    journalctl -u lighthouse-beacon -n 50 --no-pager

---

## 10. Re-check Nethermind logs

After Lighthouse is running, re-check Nethermind:

    sudo systemctl status nethermind --no-pager
    journalctl -u nethermind -n 50 --no-pager

What I am looking for:
- fewer or no repeated `Waiting for Forkchoice message from Consensus Layer` messages
- signs that Nethermind and Lighthouse are communicating through Engine API
- continued healthy process status for both services

---

## 11. Check process and ports

Check Lighthouse process:

    ps aux | grep lighthouse

Check commonly relevant ports:

    ss -tulpn | grep -E '8551|5052|9000'

Typical meaning:
- `8551` = Nethermind Engine API
- `5052` = Lighthouse REST API
- `9000` = Lighthouse P2P networking

---

## 12. Useful service commands

    sudo systemctl start lighthouse-beacon
    sudo systemctl stop lighthouse-beacon
    sudo systemctl restart lighthouse-beacon
    sudo systemctl status lighthouse-beacon --no-pager
    journalctl -u lighthouse-beacon -f
    journalctl -u lighthouse-beacon -n 50 --no-pager

---

## 13. Notes

- Lighthouse is the **consensus client**
- Nethermind is the **execution client**
- Both communicate using the **Engine API**
- The JWT secret secures that communication
- A full post-Merge Ethereum node requires both clients

---

## 14. What I learned

- how to create a JWT secret for execution-consensus communication
- how Lighthouse connects to Nethermind through `8551`
- how to install Lighthouse from a Linux release binary
- how to configure Lighthouse as a `systemd` service
- how to inspect Lighthouse logs with `journalctl`
- how execution and consensus clients fit together in a post-Merge node

---

## 15. Outcome

At the end of this setup, I should have:

- Nethermind running as the execution client
- Lighthouse running as the consensus client
- both clients connected using a shared JWT secret
- a more complete Ethereum node architecture than execution-only mode
- a better understanding of multi-service blockchain infrastructure
