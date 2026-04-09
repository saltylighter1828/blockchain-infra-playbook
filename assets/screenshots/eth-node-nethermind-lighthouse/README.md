# 📸 Ethereum Node Screenshot Evidence (Nethermind + Lighthouse)

This folder contains screenshot evidence from a live Ethereum node running on **Ubuntu under WSL2**, using:

- **Nethermind** as the **execution client**
- **Lighthouse** as the **consensus client**
- **systemd** for service orchestration
- **JWT-authenticated Engine API** communication between execution and consensus layers

These screenshots document the real operational state of the node, including service health, sync progress, peer-facing ports, execution-consensus communication, and disk growth during chain ingestion.

---

## ✅ What This Evidence Demonstrates

- Both execution and consensus clients are running under `systemd`
- Lighthouse is actively syncing and receiving beacon chain data
- Nethermind is receiving **forkchoice updates** from the consensus client
- Required ports are open and listening correctly
- Blockchain data is actively being written to disk
- The full node stack is functioning as a real post-Merge Ethereum node

---

## 🖥️ Screenshots

### 1. systemd Service Status
![systemctl status nethermind and lighthouse-beacon](./systemctl%20status%20nethermind%20and%20lighthouse-beacon.png)

Shows both `nethermind` and `lighthouse-beacon` running as active `systemd` services.

---

### 2. Lighthouse Syncing Logs
![Lighthouse syncing logs](./Lighthouse%20syncing%20logs.png)

Shows Lighthouse beacon-node sync activity, including block processing and live consensus-layer progress.

---

### 3. Nethermind Receiving Forkchoice
![Nethermind receiving forkchoice](./Nethermind%20receiving%20forkchoice.png)

Shows Nethermind receiving forkchoice updates from Lighthouse, confirming successful execution-consensus integration through the Engine API.

---

### 4. Open Ports
![Ports Open](./Ports%20Open.png)

Shows the key listening ports for the node stack, including:

- `8545` → JSON-RPC
- `8551` → Engine API
- `30303` → Nethermind P2P
- `9000` → Lighthouse P2P

---

### 5. Disk Usage Growth
![Disk usage growing](./Disk%20usage%20growing.png)

Shows blockchain data growth on disk as the node syncs and stores execution and consensus data.

---

## 🧠 Operational Context

This screenshot set supports the related documentation in this repository, including:

- Nethermind installation and service setup
- Lighthouse installation and service setup
- Execution vs consensus architecture notes
- Node operator health checks and debugging workflows

---

## 📌 Why This Matters

This is not a static or simulated setup.

These screenshots provide proof that the node is:

- running in a real Linux environment
- managed with service-level operational controls
- participating in the Ethereum network
- syncing live blockchain data
- demonstrating successful execution + consensus client coordination

---

## 🔗 Related Documentation

See the corresponding setup and operations notes in this repository for the full build and debugging workflow.
