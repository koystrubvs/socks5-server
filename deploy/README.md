# vpn-socks, SER-891

The fork keeps the upstream application and IP allowlist implementation.
Go 1.26.10 fixes the stdlib findings including CVE-2026-111943;
golang.org/x/net 0.60.0 fixes the x/net findings. Vendored modules are regenerated.
The distroless nonroot runtime is pinned to a digest.

Build a committed revision before deployment:

```sh
docker build --build-arg GOLANG_VERSION=1.26.10 -t vpn-socks-secfix:ser891-20261022 .
trivy image --scanners vuln --severity HIGH,CRITICAL --exit-code 1 vpn-socks-secfix:ser891-20261022
```

Deploy only through DevOps in the 04:00–05:00 UTC maintenance window.
Copy `deploy/docker-compose.yml` to `/home/koystrubvs/services/vpn-socks/docker-compose.yml`
after backing up the live Compose file and verifying that only its image and comments change.
Keep the live `.env` in place; do not copy credentials into this repository.
Keep `ALLOWED_IPS=10.0.1.251`, `REQUIRE_AUTH=false`, the coolify network,
the proxy address 10.0.1.250 and host policy routing unchanged.

Before replacement, retain the running image as
`serjs/go-socks5-proxy:prod-pre-SER-891` and use `docker compose up -d --no-build --pull never vpn-socks`.
The host has no published SOCKS port. Test from the allowed client's network namespace:
SOCKS5 connections must pass and use the tunnel's exit address. A different source IP
must be rejected. Check `/usr/local/sbin/vpn-socks-policy.sh status` as well.
The next scheduled Trivy scan must no longer show the old findings for the running image.

Rollback: set the image in the live Compose file to
`serjs/go-socks5-proxy:prod-pre-SER-891` and run the same Compose command.
Keep the same network, IP address and `.env`; do not stop the tunnel or policy routing.
