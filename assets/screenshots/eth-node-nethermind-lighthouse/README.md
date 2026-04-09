# Eth Node Screenshot Evidence (Nethermind + Lighthouse)

This folder contains screenshot evidence from my Ethereum node setup using:

- **Nethermind** as the execution client
- **Lighthouse** as the consensus client
- Ubuntu on WSL2
- `systemd` for service management

These screenshots document service health, syncing behavior, port exposure, disk growth, and successful execution-consensus communication.

---

## Screenshots

### Service status: Nethermind and Lighthouse Beacon
![systemctl status nethermind and lighthouse-beacon](systemctl%20status%20nethermind%20and%20lighthouse-beacon.png)

Shows both services running under `systemd`.

---

### Lighthouse syncing logs
![Lighthouse syncing logs](Lighthouse%20syncing%20logs.png)

Shows Lighthouse syncing activity and beacon-node log output.

---

### Nethermind receiving forkchoice
![Nethermind receiving forkchoice](Nethermind%20receiving%20forkchoice.png)

Shows Nethermind receiving consensus-layer forkchoice updates after Lighthouse was connected.

---

### Ports open
![Ports Open](Ports%20Open.png)

Shows the relevant listening ports for the Ethereum node stack.

---

### Disk usage growing
![Disk usage growing](Disk%20usage%20growing.png)

Shows disk usage and chain data growth as the node stores blockchain data.

---

## Notes

This folder serves as proof-of-life evidence for my Nethermind + Lighthouse node environment and supports the related setup and operations documentation in the repository.
