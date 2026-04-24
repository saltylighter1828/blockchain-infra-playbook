# Incident: JWT InvalidToken - Lighthouse/Nethermind Auth Failure

**Date:** 24 Apr 2026  
**Duration:** ~35 minutes  
**Severity:** High — nodes not communicating

## Symptoms
- Lighthouse: `ERROR Failed jwt authorization error: InvalidToken`
- Nethermind: `Not receiving ForkChoices from consensus client`

## Root Cause
Nethermind auto-generated a JWT in:
`/mnt/n/nethermind-data/keystore/jwt-secret`

While both services were configured to use:
`/secrets/jwt.hex`

Docker Desktop WSL integration restart exposed the mismatch.

## Fix
```bash
sudo cp /mnt/n/nethermind-data/keystore/jwt-secret /secrets/jwt.hex
sudo systemctl restart lighthouse-beacon
sudo rm /mnt/n/nethermind-data/keystore/jwt-secret
```

## Prevention
Added JWT existence check to node-check.sh
