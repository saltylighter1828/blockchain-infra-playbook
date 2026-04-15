# 🌐 Networking Basics

A concise reference for the networking concepts most relevant to Ethereum node operations, service communication, and monitoring.

---

## Why Networking Matters

Running a node is not only about starting services. It also means understanding how those services communicate:

- execution and consensus clients exchange data over local endpoints
- peers connect over the network to sync chain data
- monitoring systems scrape metrics over HTTP
- RPC endpoints expose node data for inspection and tooling

At a practical level, networking helps answer questions like:

- which service is listening on which port?
- is an endpoint bound to localhost or exposed more broadly?
- are two services actually able to talk to each other?
- is a monitoring target reachable?

---

## ⚔️ TCP vs UDP

### TCP
TCP is connection-oriented and designed for reliable communication.

Key characteristics:

- reliable delivery
- ordered packets
- retransmission if packets are lost
- better suited to APIs, web traffic, and service-to-service communication

Common node-related examples:

- JSON-RPC
- Engine API
- Prometheus scraping
- Grafana web interface

### UDP
UDP is connectionless and lightweight.

Key characteristics:

- faster and lower overhead
- no guarantee of delivery
- no guarantee of order
- useful where speed matters more than reliability

Common node-related examples:

- some peer-to-peer discovery traffic
- lightweight network signalling

### Practical Rule
Use TCP when you care about correctness and complete delivery.  
Use UDP when you care about speed and can tolerate loss.

---

## 🔌 Key Ports in This Stack

These are the main ports used in the local Ethereum node environment.

- `8545` → JSON-RPC endpoint for querying the execution client
- `8551` → Engine API used by the consensus client to talk to the execution client
- `30303` → Nethermind peer-to-peer networking
- `9000` → Lighthouse peer-to-peer networking
- `5054` → Lighthouse metrics
- `6060` → Nethermind metrics
- `9100` → Node Exporter metrics
- `9090` → Prometheus
- `3000` → Grafana

### Port Roles

#### `8545` JSON-RPC
Used to query the execution client directly.

Typical use cases:

- checking block number
- querying sync state
- retrieving chain data
- testing node responsiveness

#### `8551` Engine API
Used internally between Lighthouse and Nethermind.

Important notes:

- not a public query endpoint
- usually bound to `127.0.0.1`
- protected with JWT authentication
- critical for execution ↔ consensus coordination

#### `30303` and `9000` P2P
Used for peer-to-peer communication.

These ports support:

- peer discovery
- block and chain data exchange
- network participation

#### Metrics Ports
Used for observability rather than chain interaction.

- `5054` → Lighthouse metrics
- `6060` → Nethermind metrics
- `9100` → host metrics via Node Exporter
- `9090` → Prometheus query and scrape management
- `3000` → Grafana dashboards

---

## 📡 RPC

RPC stands for **Remote Procedure Call**.

It is a way for one system to request information or actions from another system as though it were calling a local function.

In node operations, RPC is commonly used to:

- query chain state
- retrieve block numbers
- check sync progress
- inspect peers
- test whether a node is responding

In simple terms:

> RPC is how you ask a node to do something or tell you something.

---

## 📦 JSON-RPC

JSON-RPC is a lightweight RPC protocol that uses JSON as the message format.

Ethereum execution clients commonly expose a JSON-RPC interface for tools, scripts, dashboards, and manual checks.

### Common Example

Requesting the current execution block number:

    curl -s -H "Content-Type: application/json" \
      -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
      http://127.0.0.1:8545

This sends a JSON-RPC request to the execution client asking for `eth_blockNumber`.

### What This Tells You
- whether the endpoint is reachable
- whether the node is responding correctly
- the current block height known to the execution client

---

## Localhost vs Exposed Interfaces

A very important networking concept for node operations is the difference between:

- `127.0.0.1` / `localhost`
- `0.0.0.0` or externally reachable interfaces

### `127.0.0.1`
Means the service is only reachable from the local machine.

Use this for:
- Engine API
- local metrics
- private management endpoints

### `0.0.0.0`
Means the service is listening on all interfaces.

Use with caution, because it may expose a service beyond the local system.

### Practical Security Principle
Sensitive endpoints should stay bound to localhost unless there is a deliberate reason to expose them.

That is especially important for:
- Engine API
- JSON-RPC
- metrics endpoints

---

## Useful Commands

### Check Listening Ports

    ss -tulpn

### Filter Important Node Ports

    ss -tulpn | grep -E '8545|8551|30303|9000|5054|6060|9100|9090|3000'

### Test JSON-RPC Reachability

    curl -s -H "Content-Type: application/json" \
      -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
      http://127.0.0.1:8545

### Test Prometheus Health

    curl -s http://127.0.0.1:9090/-/healthy

### Test Grafana Health

    curl -s http://127.0.0.1:3000/api/health

### Test Lighthouse Metrics

    curl -s http://127.0.0.1:5054/metrics | head

### Test Nethermind Metrics

    curl -s http://127.0.0.1:6060/metrics | head

---

## Healthy Networking Looks Like

A healthy node networking setup usually means:

- expected ports are listening
- local-only endpoints remain bound to `127.0.0.1`
- P2P ports are active
- JSON-RPC responds successfully
- Engine API is reachable by the consensus client
- metrics endpoints are reachable by Prometheus
- Prometheus scrape targets remain up

---

## Common Networking Failure Signs

Watch for:

- missing listening ports
- connection refused errors
- timeouts on local endpoints
- metrics endpoints not responding
- execution and consensus not communicating
- JSON-RPC unreachable
- Prometheus targets showing down
- services bound to the wrong interface

---

## Key Takeaway

For blockchain infrastructure work, networking is not abstract background knowledge. It is part of daily operations.

Understanding ports, protocols, endpoint scope, and service communication makes it much easier to:

- verify node health
- debug service interaction
- secure sensitive endpoints
- confirm monitoring coverage
- operate the stack with confidence
