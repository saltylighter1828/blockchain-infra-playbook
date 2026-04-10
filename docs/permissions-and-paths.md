## JWT Permissions and WSL Path Validation

During my Nethermind + Lighthouse node setup, I verified that both clients could securely access the shared JWT secret used for Engine API authentication.

I created a shared `jwtreaders` group, set `/secrets/jwt.hex` to `root:jwtreaders` with `640` permissions, and confirmed both `nethermind` and `lighthouse` could read it without making the secret world-readable.

I also hardened the Nethermind data path by restricting `/home/nethermind` and `/home/nethermind/data` to the `nethermind` service account, while confirming Lighthouse could still write to `/var/lib/lighthouse`.

Finally, I confirmed the active WSL2 distro was `UbuntuNode` with BasePath `N:\WSL\UbuntuNode`, which verified that the node was running in the correct Linux environment on the N: drive rather than a different Ubuntu distro on C:.

This check confirmed correct JWT wiring, service permissions, data directory access, and WSL path placement for both systemd-managed clients.
