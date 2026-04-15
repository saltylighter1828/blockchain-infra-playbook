# Consensus Client Notes

This document captures a practical understanding of Ethereum consensus clients, why an execution client alone is not sufficient for a complete post-Merge node, what port `8551` is used for, and why Lighthouse is the consensus client paired with Nethermind in this setup.

---

## Purpose

The current environment uses:

- **Nethermind** as the execution client
- Ubuntu on WSL2
- `systemd` for service management

Nethermind is operating correctly as a service, but its logs include messages such as:

- `Waiting for Forkchoice message from Consensus Layer`
- `Not receiving ForkChoices from the consensus client that are required to sync`

Those messages do not indicate that Nethermind is broken. They indicate that the execution layer is healthy, but still waiting for a consensus client to provide the forkchoice updates required for full post-Merge operation. :contentReference[oaicite:0]{index=0}

---

## What a Consensus Client Does

A consensus client is responsible for the **consensus layer** of Ethereum.

Its role includes:

- following the Beacon Chain
- tracking slots and epochs
- validating consensus-layer data
- determining the canonical chain head
- sending forkchoice updates to the execution client
- participating in proof-of-stake networking

A consensus client does **not** execute transactions or run EVM smart contract logic directly.

Instead, it works in partnership with an execution client to form a complete Ethereum node.

Common consensus clients include:

- Lighthouse
- Teku
- Prysm
- Nimbus
- Lodestar

---

## What Nethermind Does on Its Own

Nethermind is an **execution client**.

Its role includes:

- processing Ethereum transactions
- executing EVM smart contract logic
- maintaining the execution-layer state database
- exposing JSON-RPC endpoints
- serving blockchain data to tools and applications
- participating in execution-layer peer-to-peer networking

In the current setup, Nethermind is already handling:

- JSON-RPC on port `8545`
- Engine API on port `8551`
- P2P networking on port `30303`

That means Nethermind is functioning correctly as the execution side of the node. What is still missing is the consensus side. :contentReference[oaicite:1]{index=1}

---

## Why Nethermind Waits for Forkchoice

After Ethereum moved to proof of stake, the execution client no longer decides chain head and finality on its own.

That responsibility now sits with the consensus client, which informs the execution client which head is valid through **forkchoice updates**.

That is why Nethermind logs messages such as:

- `Waiting for Forkchoice message from Consensus Layer`
- `Not receiving ForkChoices from the consensus client that are required to sync`

In practice, those messages mean:

- Nethermind is running
- execution-layer networking is active
- the execution client is waiting for consensus-layer guidance
- a consensus client still needs to be connected

So at this stage, the setup represents a healthy execution client, but not yet a complete post-Merge Ethereum node.

---

## What Port `8551` Is For

Port `8551` is used for the **Engine API**.

The Engine API is the private communication bridge between:

- the **execution client** (`Nethermind`)
- the **consensus client** (`Lighthouse` in this setup)

Through this interface, the consensus client can:

- send forkchoice updates
- coordinate payload building
- request execution-related operations
- align execution with consensus decisions

In practical terms:

- `8545` is primarily for JSON-RPC
- `30303` is for execution-layer peer-to-peer networking
- `8551` is the execution ↔ consensus bridge

This is why `8551` is a critical port in any post-Merge Ethereum node.

---

## Why a Consensus Client Is Required

An execution client alone is no longer enough to operate a full Ethereum node.

Without a consensus client:

- the execution layer can run
- JSON-RPC can be available
- P2P networking can be active
- local service health can look normal

But the node is still incomplete because:

- chain head and finality are not being driven by the consensus layer
- forkchoice updates are missing
- the execution client cannot fully participate as part of a complete post-Merge stack

The consensus client is therefore not an optional add-on. It is part of the full node architecture.

---

## Chosen Consensus Client: Lighthouse

The consensus client selected for this setup is:

- **Lighthouse**

### Why Lighthouse

Reasons for choosing Lighthouse include:

- it is a well-established Ethereum consensus client
- it is widely used and well documented
- it is a good fit for Linux and `systemd`-based learning
- it makes the execution ↔ consensus split easier to understand in practice
- it fits naturally into an infrastructure-focused learning path

Lighthouse is the next step in turning a working Nethermind execution client into a complete Ethereum node stack.

---

## Why This Matters for Infrastructure Learning

Adding a consensus client matters for much more than simply “installing more software”.

It introduces several important infrastructure concepts:

- execution vs consensus separation
- multi-service node architecture
- service-to-service communication
- Engine API relationships
- JWT-secured authentication between services
- new logging, monitoring, and troubleshooting paths
- the operational model of a complete post-Merge Ethereum node

This is one of the clearest examples of how blockchain infrastructure is composed from multiple cooperating services rather than a single process.

---

## Current Understanding

At this stage, the working understanding is:

- Nethermind is the execution client
- a consensus client is still required for a complete post-Merge node
- Nethermind waits for forkchoice because no consensus client is connected yet
- port `8551` is the Engine API bridge between execution and consensus
- Lighthouse is the consensus client paired with Nethermind in this environment

---

## Next Step

The next operational step is to:

- install Lighthouse
- connect Lighthouse to Nethermind
- verify Engine API communication over `8551`
- confirm that execution and consensus logs reflect successful coordination
- observe how node behaviour changes once the full stack is complete

---

## Key Takeaways

- a consensus client manages Ethereum’s consensus layer
- Nethermind alone is not a complete post-Merge node
- forkchoice messages are essential because consensus now determines the canonical head
- port `8551` is the Engine API bridge between execution and consensus
- Lighthouse is the chosen consensus client for completing the node stack

---

## Outcome

This note provides a clearer mental model of the execution-consensus split before moving into Lighthouse installation and integration.

It explains:

- what is already working
- what is still missing
- why the missing component matters
- how the full Ethereum node architecture fits together
