# Consensus Client Notes

This document records my current understanding of Ethereum consensus clients, why Nethermind alone is not enough to form a complete post-Merge node, what port `8551` is used for, and which consensus client I plan to pair with Nethermind next.

---

## Purpose

My current setup uses:

- **Nethermind** as the execution client
- Ubuntu on WSL2
- `systemd` for service management

Nethermind is running correctly as a service, but its logs show messages such as:

- `Waiting for Forkchoice message from Consensus Layer`
- `Not receiving ForkChoices from the consensus client that are required to sync`

These messages indicate that the execution client is healthy, but it is still waiting for a consensus client to provide forkchoice updates.

---

## What is a consensus client?

A consensus client is the part of an Ethereum node that handles the **consensus layer**.

Its responsibilities include:

- following the Beacon Chain
- tracking slots and epochs
- validating consensus data
- determining the canonical chain head
- sending forkchoice updates to the execution client
- participating in proof-of-stake consensus networking

A consensus client does **not** execute transactions or smart contracts directly.

Instead, it works together with an execution client.

Examples of Ethereum consensus clients include:

- Lighthouse
- Teku
- Prysm
- Nimbus
- Lodestar

---

## What is Nethermind doing on its own?

Nethermind is an **execution client**.

Its responsibilities include:

- processing Ethereum transactions
- executing EVM smart contract logic
- maintaining the execution-layer state database
- exposing JSON-RPC endpoints
- serving blockchain data to tools and applications
- participating in execution-layer peer-to-peer networking

In my current setup, Nethermind is already handling:

- JSON-RPC on port `8545`
- Engine API on port `8551`
- P2P networking on port `30303`

This means Nethermind is working correctly as the execution side of the node.

---

## Why does Nethermind wait for forkchoice?

After Ethereum moved to proof of stake, the execution client no longer decides chain head and finality by itself.

Instead, the consensus client tells the execution client which chain head is valid through **forkchoice updates**.

That is why Nethermind logs messages like:

- `Waiting for Forkchoice message from Consensus Layer`
- `Not receiving ForkChoices from the consensus client that are required to sync`

These messages do **not** necessarily mean Nethermind is broken.

They mean:

- Nethermind is running
- peer networking is active
- the execution layer is waiting for the consensus layer
- a consensus client still needs to be connected

So at this stage, I have an execution client running, but not yet a complete post-Merge Ethereum node.

---

## What is port `8551` for?

Port `8551` is used for the **Engine API**.

The Engine API is the communication bridge between:

- the **execution client** (Nethermind)
- the **consensus client** (for me, planned: Lighthouse)

This API allows the consensus client to:

- send forkchoice updates
- request payload building
- coordinate block execution with the execution client

In practical terms:

- `8545` is mainly for JSON-RPC
- `30303` is for peer-to-peer networking
- `8551` is the private execution ↔ consensus connection

This is why `8551` matters so much in a post-Merge node.

---

## Which consensus client do I want to pair with Nethermind?

I plan to pair Nethermind with:

- **Lighthouse**

### Why Lighthouse?

My reasons:

- it is a well-known Ethereum consensus client
- it is popular and widely used
- it seems beginner-friendly for learning node architecture
- it is a good fit for a Linux/systemd learning path
- it will help me understand how a full Ethereum node fits together

I want to use Lighthouse as the next step in turning my current Nethermind setup into a more complete node stack.

---

## Why this matters for infrastructure learning

Adding a consensus client is important because it teaches me:

- execution vs consensus separation
- multi-service architecture
- service-to-service communication
- Engine API relationships
- additional logs and troubleshooting paths
- how a complete Ethereum node is assembled and operated

This is not just about “installing more software”.
It is about understanding how distributed blockchain infrastructure is composed.

---

## Current understanding

Right now, my understanding is:

- Nethermind is the execution client
- a consensus client is still required
- Nethermind waits for forkchoice because no consensus client is connected yet
- port `8551` is the Engine API bridge between the two
- Lighthouse is the consensus client I want to learn next

---

## Next step

My next planned step is to:

- install Lighthouse
- connect Lighthouse to Nethermind
- verify that the execution and consensus clients can communicate correctly
- observe how the logs and node behavior change once the node stack is complete

---

## What I learned

- what a consensus client is
- why Nethermind alone is not a complete post-Merge node
- why forkchoice messages matter
- what port `8551` is used for
- why Lighthouse is my chosen next client

---

## Outcome

This note gives me a clear mental model before I move on to installing Lighthouse.

It helps me explain:

- what is currently working
- what is still missing
- what the next step is
- why adding a consensus client is necessary
