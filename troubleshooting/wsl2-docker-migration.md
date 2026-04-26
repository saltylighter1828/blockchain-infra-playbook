# Incident: WSL2 Docker Interference + Full Migration

**Date:** 24-26 Apr 2026  
**Duration:** ~2 days  
**Severity:** High  

## Symptoms
- Lighthouse: `ERROR Failed jwt authorization error: InvalidToken`
- Nethermind: `Not receiving ForkChoices from consensus client`
- C: drive at 99% full (17GB free)

## Root Causes

### Issue 1 - JWT Mismatch
Nethermind auto-generated JWT at:
`/mnt/n/nethermind-data/keystore/jwt-secret`
While Lighthouse read from:
`/secrets/jwt.hex`
Docker WSL restart exposed the mismatch.

### Issue 2 - Duplicate WSL Environments
Discovered two WSL distros running simultaneously:
- Ubuntu (partial, only Nethermind)
- UbuntuNode (complete, all services)
Both accessing same Nethermind DB on N: drive.

### Issue 3 - C: Drive 99% Full
Hidden culprit: WSL2 virtual disk (ext4.vhdx) 
sitting on C: drive taking 563GB.

## Fixes

### JWT Fix
```bash
sudo cp /mnt/n/nethermind-data/keystore/jwt-secret /secrets/jwt.hex
sudo systemctl restart lighthouse-beacon
sudo diff /secrets/jwt.hex /mnt/n/nethermind-data/keystore/jwt-secret
sudo rm /mnt/n/nethermind-data/keystore/jwt-secret
```

### WSL Migration (C: → N: drive)
```powershell
# Export
wsl --export UbuntuNode "N:\ubuntunode-backup.tar"

# Unregister from C:
wsl --unregister UbuntuNode

# Import to N:
wsl --import UbuntuNode "N:\WSL\UbuntuNode" "N:\ubuntunode-backup.tar" --version 2

# Fix default user
wsl -d UbuntuNode -u root -- bash -c "echo -e '[user]\ndefault=saltylighter' >> /etc/wsl.conf"
```

## C: Drive Recovery
Before:   17GB free (99% full)
After:    660GB free
Recovered:

Ubuntu distro unregistered:   560GB
Docker Desktop uninstalled:    18GB
Files moved to N:              96GB

## Prevention
- Uninstalled Docker Desktop completely
- Single JWT source: /secrets/jwt.hex
- WSL virtual disk moved to N: (T710 NVMe)
- Added JWT check to node-check.sh

## Lessons Learned
- Always verify JWT source on both clients after restarts
- Docker Desktop WSL integration interferes with systemd
- Keep WSL virtual disks on dedicated fast drives
- Document incidents in real time
