# Nethermind Storage Incident: Root Disk Filled by Duplicate Mainnet DB

Resolved a storage incident where the Linux root filesystem `/` became critically full because a duplicate Nethermind mainnet database was stored under `/home/nethermind/data`, while the intended active database location was on `N:` at `/mnt/n/nethermind-data`. After updating the systemd service to point to the correct data directory and removing the duplicate database from root storage, both Nethermind and Lighthouse returned to a healthy syncing state.

## Environment
- **Execution client:** Nethermind
- **Consensus client:** Lighthouse
- **Platform:** Ubuntu on WSL2
- **Monitoring:** Prometheus + Grafana
- **Large storage target:** `N:` mounted at `/mnt/n`

## Problem
The node became unstable the morning after syncing overnight.

### Symptoms observed
- `nethermind.service` was failing and repeatedly restarting
- `lighthouse-beacon.service` was not remaining in a steady healthy state
- Lighthouse showed execution engine connection issues
- Nethermind had a very high restart counter
- The root filesystem `/` was nearly full

## Initial checks
Used the following commands to inspect service health, logs, and machine resources:

    systemctl status lighthouse-beacon nethermind --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager
    journalctl -u nethermind -n 20 --no-pager
    uptime
    free -h
    df -h

### Key finding
The root disk was critically full:

    /dev/sdd  1007G  942G   14G  99%  /

This identified storage exhaustion as the likely root cause rather than CPU or memory pressure.

## Investigation
To find what was consuming disk space, the filesystem was inspected with:

    sudo du -xh / --max-depth=1 2>/dev/null | sort -h
    sudo du -xh /home /var --max-depth=2 2>/dev/null | sort -h

This showed that the major consumer was:

    /home/nethermind/data = 930G

A deeper check was then run:

    sudo du -xh /home/nethermind/data --max-depth=2 2>/dev/null | sort -h | tail -n 30

This confirmed that:

    /home/nethermind/data/nethermind_db/mainnet = 930G

showing that the root disk was being filled by the Nethermind mainnet database.

## Why this was unexpected
The intended storage location for Nethermind was the much larger `N:` drive. To verify the actual configured data path, the systemd service was checked with:

    systemctl cat nethermind
    sudo systemctl show nethermind -p FragmentPath -p ExecStart

The actual configured path still contained:

    --data-dir /home/nethermind/data

So although `N:` had plenty of free space, Nethermind was actually writing to the Linux root filesystem.

## Additional discovery: duplicate database
The intended `N:` path was also checked with:

    sudo du -xh /mnt/n/nethermind-data --max-depth=2 2>/dev/null | sort -h | tail -n 30

This revealed that there was already another full Nethermind database stored on `N:`:

    /mnt/n/nethermind-data/nethermind_db/mainnet = 931G

At that point it became clear that there were effectively **two giant Nethermind databases**:

- Wrong path on root disk:
  - `/home/nethermind/data/nethermind_db/mainnet`
- Intended path on `N:`:
  - `/mnt/n/nethermind-data/nethermind_db/mainnet`

The root-disk copy was the one causing the outage.

## Root cause
The incident was caused by a **duplicate Nethermind mainnet database stored on the Linux root filesystem**, combined with the Nethermind systemd service still pointing to the wrong data directory.

### Root cause statement
- Nethermind was configured to use `/home/nethermind/data`
- A valid large database also existed on `/mnt/n/nethermind-data`
- The duplicate root-disk database filled `/`
- Nethermind became unstable and entered a restart loop
- Lighthouse was affected because its execution client dependency was unhealthy

## Fix

### 1. Updated the Nethermind service to use the `N:` drive
Edited the service file so `ExecStart` used the correct data directory:

    ExecStart=/usr/bin/nethermind --data-dir /mnt/n/nethermind-data --jsonrpc-enabled true --jsonrpc-enginehost 127.0.0.1 --jsonrpc-engineport 8551 --jsonrpc-jwtsecretfile /secrets/jwt.hex

After editing the service file, systemd was reloaded with:

    sudo systemctl daemon-reload

The active service definition was then verified with:

    sudo systemctl show nethermind -p FragmentPath -p ExecStart

### 2. Confirmed the `N:` target path was writable
Practical write access to the `N:` target path was confirmed using:

    sudo -u nethermind touch /mnt/n/nethermind-data/.write_test && echo writable || echo NOT_writable

### 3. Stopped both clients
Both clients were then stopped:

    sudo systemctl stop lighthouse-beacon
    sudo systemctl stop nethermind

### 4. Removed the duplicate root-disk database
The duplicate root-disk database was removed with:

    sudo rm -rf /home/nethermind/data/nethermind_db

This immediately freed the Linux root filesystem.

### 5. Restarted Nethermind and Lighthouse
Nethermind and Lighthouse were then restarted:

    sudo systemctl start nethermind
    sudo systemctl start lighthouse-beacon

## Verification

### Disk verification
After cleanup, root storage returned to a healthy state:

    /dev/sdd  1007G   12G  944G   2%  /

The active `N:` drive still had plenty of free space:

    N:\  3.7T  2.0T  1.8T  53%  /mnt/n

### Service verification
Nethermind was confirmed to be running from the correct path using the `N:` drive. Healthy post-fix indicators included:
- Nethermind receiving `ForkChoice`
- Nethermind processing new blocks
- Nethermind logging `Synced Chain Head to ...`
- Lighthouse showing `Synced`
- Lighthouse peer count rising into the 70s
- Lighthouse execution hashes being marked as `verified`

## Commands used during troubleshooting

### Service and log checks
    systemctl status lighthouse-beacon nethermind --no-pager
    journalctl -u lighthouse-beacon -n 20 --no-pager
    journalctl -u nethermind -n 20 --no-pager

### System resource checks
    uptime
    free -h
    df -h

### Disk usage investigation
    sudo du -xh / --max-depth=1 2>/dev/null | sort -h
    sudo du -xh /home /var --max-depth=2 2>/dev/null | sort -h
    sudo du -xh /home/nethermind/data --max-depth=2 2>/dev/null | sort -h | tail -n 30
    sudo du -xh /mnt/n/nethermind-data --max-depth=2 2>/dev/null | sort -h | tail -n 30

### Service configuration checks
    systemctl cat nethermind
    sudo systemctl show nethermind -p FragmentPath -p ExecStart

### Storage structure inspection
    tree -L 3 /mnt/n/nethermind-data

## What I learned
- `df -h` tells me **how full a filesystem is**
- `du -sh` and `du -xh` tell me **which directories are consuming space**
- Parent directories can show similar large sizes because they include child folder contents
- Mounted storage is only useful if the service is actually configured to write there
- WSL-mounted Windows drives can behave differently with ownership, but write testing confirms practical usability
- A healthy troubleshooting flow is:
  1. check service status
  2. check logs
  3. check system health
  4. identify the pressure point
  5. trace the exact path
  6. fix config
  7. verify recovery

## Outcome
The incident was resolved successfully.

### Final state
- Root disk pressure was removed
- Nethermind now runs from `/mnt/n/nethermind-data`
- The duplicate root-disk database was removed
- Lighthouse and Nethermind both returned to healthy syncing
- The node resumed normal execution and consensus coordination

## Next improvements
- Add a Grafana panel or alert for root disk usage
- Add a written node runbook for morning health checks
- Document the expected Nethermind data path clearly in setup notes
- Consider enabling Nethermind metrics later for client-specific observability
