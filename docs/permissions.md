# Permissions and Access Hardening
## Nethermind + Lighthouse JWT and Data Directory Checks

This document records the permission checks and hardening steps used to securely run a local Ethereum node with:

- **Nethermind** as the execution client
- **Lighthouse** as the consensus client
- **systemd** for service management

The goal was to make sure:

- both clients could read the shared JWT secret
- secrets were not exposed to all users
- service data directories were writable only by the correct service account
- node data was not world-readable unless required

## Why this matters

When Lighthouse connects to Nethermind over the Engine API, both clients must use the same JWT secret for authenticated communication.

If the JWT file is too restricted, one of the services will fail to read it.

If it is too open, the secret is unnecessarily exposed.

This setup follows a **least privilege** approach:

- only the required service users can read the JWT secret
- only the required service user can access execution client data
- systemd service files remain root-owned
- data and secrets stay locked down

## JWT secret setup

The JWT secret was stored at:

`/secrets/jwt.hex`

Final desired state:

- `/secrets` directory exists
- `jwt.hex` contains exactly 64 hex characters
- file ownership is `root:jwtreaders`
- file permissions are `640`
- both `nethermind` and `lighthouse` are members of the `jwtreaders` group

## Problem discovered

Initially, the JWT secret file did not exist.

After it was created, it was readable only by `root`, which would prevent the service users from using it.

A shared group approach was used so the file did not need to be made public.

## Final JWT permissions model

### Create the shared readers group

    sudo groupadd jwtreaders
    sudo usermod -aG jwtreaders nethermind
    sudo usermod -aG jwtreaders lighthouse

### Create a clean 64-byte JWT secret

    sudo mkdir -p /secrets
    sudo sh -c 'printf %s "$(openssl rand -hex 32)" > /secrets/jwt.hex'

### Assign ownership and permissions

    sudo chown root:jwtreaders /secrets/jwt.hex
    sudo chmod 640 /secrets/jwt.hex

### Verify final state

    ls -ld /secrets
    ls -l /secrets/jwt.hex
    sudo wc -c /secrets/jwt.hex
    getent group jwtreaders
    id nethermind
    id lighthouse

Expected result:

    drwxr-xr-x 2 root root 4096 /secrets
    -rw-r----- 1 root jwtreaders 64 /secrets/jwt.hex
    64 /secrets/jwt.hex
    jwtreaders:x:1004:nethermind,lighthouse

## JWT access validation

The most important check was not just file ownership, but whether the service users could actually read the secret.

### Verification commands

    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_can_read_jwt || echo nethermind_cannot_read_jwt
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_can_read_jwt || echo lighthouse_cannot_read_jwt

### Expected result

    nethermind_can_read_jwt
    lighthouse_can_read_jwt

This confirmed that both systemd-managed services could access the Engine API JWT secret without exposing it to all users.

## Checking path permissions

`namei -l` was used to inspect every directory in the path, not just the final file.

### Commands

    namei -l /secrets/jwt.hex
    namei -l /home/nethermind/data
    namei -l /var/lib/lighthouse

This is useful for identifying whether a parent directory blocks traversal even if the target file or folder itself looks correct.

## Nethermind home and data directory hardening

The execution client data directory was initially more open than necessary.

Initial state:

    /home/nethermind/data -> drwxrwxr-x

This allowed other users to traverse and read directory listings, which was not needed.

### Final hardening applied

    sudo chown -R nethermind:nethermind /home/nethermind
    sudo chmod 750 /home/nethermind
    sudo chmod 700 /home/nethermind/data

### Final state

- `/home/nethermind` -> `750`
- `/home/nethermind/data` -> `700`

This means:

- the `nethermind` user has full access
- no other user can access the data directory
- the service can still run normally under `systemd`

### Verification

    sudo ls -ld /home/nethermind
    sudo ls -ld /home/nethermind/data
    sudo namei -l /home/nethermind/data
    sudo -u nethermind test -w /home/nethermind/data && echo nethermind_data_ok || echo nethermind_data_bad

Expected shape:

    drwxr-x--- nethermind nethermind /home/nethermind
    drwx------ nethermind nethermind /home/nethermind/data
    nethermind_data_ok

## Lighthouse data directory check

The Lighthouse data directory was also checked to ensure the service account could write to it.

### Verification

    ls -ld /var/lib/lighthouse
    namei -l /var/lib/lighthouse
    sudo -u lighthouse test -w /var/lib/lighthouse && echo lighthouse_data_ok || echo lighthouse_data_bad

Expected result:

    drwxr-xr-x lighthouse lighthouse /var/lib/lighthouse
    lighthouse_data_ok

If desired, this directory can be hardened further later, but the current setup was functioning correctly for the service.

## systemd service file ownership

The service unit files remained root-owned and world-readable, which is normal.

### Verification

    ls -l /etc/systemd/system/nethermind.service
    ls -l /etc/systemd/system/lighthouse-beacon.service

Expected result:

    -rw-r--r-- 1 root root ... /etc/systemd/system/nethermind.service
    -rw-r--r-- 1 root root ... /etc/systemd/system/lighthouse-beacon.service

This is standard for systemd unit files.

## systemd service access model

Both clients were configured to run under dedicated service users:

- `nethermind.service` runs as `User=nethermind`
- `lighthouse-beacon.service` runs as `User=lighthouse`

This means the permission model must be built around what those specific users can read and write, rather than what the normal login user can access.

## Final security posture

After the checks and hardening steps:

- the JWT secret is not world-readable
- only `nethermind` and `lighthouse` can read the shared JWT secret through the `jwtreaders` group
- Nethermind data is restricted to the `nethermind` user
- Lighthouse data is writable by the `lighthouse` user
- service files remain properly managed by `root`
- both clients continue to run successfully under `systemd`

## Useful command pack

    ls -ld /secrets
    ls -l /secrets/jwt.hex
    sudo wc -c /secrets/jwt.hex
    getent group jwtreaders
    id nethermind
    id lighthouse
    namei -l /secrets/jwt.hex
    namei -l /home/nethermind/data
    namei -l /var/lib/lighthouse
    sudo -u nethermind test -r /secrets/jwt.hex && echo nethermind_jwt_ok || echo nethermind_jwt_bad
    sudo -u lighthouse test -r /secrets/jwt.hex && echo lighthouse_jwt_ok || echo lighthouse_jwt_bad
    sudo -u nethermind test -w /home/nethermind/data && echo nethermind_data_ok || echo nethermind_data_bad
    sudo -u lighthouse test -w /var/lib/lighthouse && echo lighthouse_data_ok || echo lighthouse_data_bad
    ls -l /etc/systemd/system/nethermind.service
    ls -l /etc/systemd/system/lighthouse-beacon.service
    sudo systemctl status nethermind --no-pager -l
    sudo systemctl status lighthouse-beacon --no-pager -l

## Summary

This permission model allows Nethermind and Lighthouse to communicate securely over the Engine API while keeping secrets and node data appropriately restricted.

The key ideas were:

- use a shared group for the JWT secret
- verify access as the real service users
- restrict data directories to the service account that owns them
- avoid unnecessary world-readable access
- confirm everything still works after hardening

This approach keeps the node functional while improving security hygiene and documenting a real-world operator troubleshooting workflow.
