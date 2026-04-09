# ETH Node: Nethermind + Lighthouse (Screenshots)

This folder contains proof-of-work screenshots from a live Ethereum node running:

- **Nethermind** (execution client)
- **Lighthouse** (consensus client)
- Connected via **Engine API (JWT authentication)**
- Managed using **systemd**

---

## 📸 What’s Included

### 🧠 Execution + Consensus Running
![Execution + Consensus Running](./systemctl%20status%20nethermind%20and%20lighthouse-beacon.png)

→ Both services active and managed by systemd

---

### 🔗 Engine API Communication
![Nethermind Forkchoice](./Nethermind%20receiving%20forkchoice.png)

→ Nethermind receiving forkchoice updates from Lighthouse  
→ Confirms execution + consensus integration is working

---

### 🌐 Networking
![Ports Open](./Ports%20Open.png)

→ Required ports open and listening:
- `8545` (JSON-RPC)
- `8551` (Engine API)
- `30303` (Execution P2P)
- `9000` (Consensus P2P)

---

### 🧠 Consensus Sync
![Lighthouse Syncing](./Lighthouse%20syncing%20logs.png)

→ Beacon node syncing, receiving blocks, and participating in the network

---

### 💾 Execution Sync Progress
![Disk Usage Growing](./Disk%20usage%20growing.png)

→ Nethermind data directory increasing in size  
→ Confirms active blockchain state sync

---

## 🚀 Summary

These screenshots demonstrate:

- A fully running **post-Merge Ethereum node**
- Proper separation of execution and consensus layers
- Successful **JWT-authenticated Engine API communication**
- Active peer connections and ongoing sync
- Real disk growth from blockchain data ingestion

---

## 🧠 Notes

- Node is running on **WSL Ubuntu with systemd enabled**
- Services are configured with:
  - auto-start (`systemctl enable`)
  - auto-restart (`Restart=on-failure`)
- Setup includes proper **Linux user separation** and **secure JWT handling**

---

## 🔥 Why This Matters

This is not a simulated environment.

This node is:
- actively syncing Ethereum mainnet
- processing real blocks
- participating in the live network
