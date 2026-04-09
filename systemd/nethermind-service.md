# ⚙️ Nethermind systemd Service

## 🔄 Reload systemd
sudo systemctl daemon-reload
- Reload service files after changes

---

## 🚀 Start service
sudo systemctl start nethermind
- Start Nethermind node

---

## 🔁 Enable on boot
sudo systemctl enable nethermind
- Automatically start on system boot

---

## 📊 Check status
systemctl status nethermind
- Verify if service is running

---

## 📜 View logs
journalctl -u nethermind -f
- View live logs from Nethermind
