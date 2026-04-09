# 🐧 Linux Commands

## 🔍 Process Check
ps aux | grep nethermind
- Check if Nethermind process is running
- Ignore the "grep" line itself

---

## 🌐 Port Check
ss -tulpn | grep -E '8545|8551|30303'
- Verify node ports are open
- 8545 → JSON-RPC
- 8551 → Engine API
- 30303 → P2P network

---

## 💾 Disk Space
df -h
- Check available disk space
- Prevent node crashes due to full disk

---

## 📦 Data Size
du -sh /home/nethermind/data
- Check size of blockchain data
- Helps monitor sync progress
