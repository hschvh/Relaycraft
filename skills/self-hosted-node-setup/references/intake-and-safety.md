# Intake and Safety

Use this file when you need to classify the topology or collect only the minimum non-sensitive facts before any deployment.

## Topology branches

1. VPS has its own public IP and the residential service is separate.
2. Residential service is an upstream HTTP/HTTPS/SOCKS5 proxy.
3. Residential service is a tunnel or routed delivery such as GRE, WireGuard, or BGP.

If the information is incomplete, stop and ask for the missing topology facts. Do not guess.

## Minimum intake

| Need | Safe form |
| --- | --- |
| VPS system role | "Ubuntu VPS", "Debian VPS", or similar |
| Public address | `<VPS_PUBLIC_IP>` or `<VPS_HOSTNAME>` |
| Desired protocol | VLESS/REALITY or Hysteria2 |
| Residential type | HTTP, HTTPS, SOCKS5, or tunnel/routing |
| Authentication form | username/password, token-less, or other non-secret category |
| UDP support | unknown until verified |
| Existing services | list names only, no configs |

Ask for secrets only through interactive input or other restricted handling. Never ask the user to paste credentials into chat.

## Stop conditions

- The user has not authorized the VPS or client environment.
- The topology is unclear enough that the next step could overwrite an existing service.
- A required capability such as UDP, certificate access, or port availability is unverified.
- The requested action would reveal or store sensitive values in the repository.

## Routing rule

When the residential resource is an upstream proxy, the client `server` field still points at the VPS address. The residential proxy is configured as upstream egress, not as the client endpoint.

## Output habit

State the topology, the non-sensitive facts gathered, what remains unknown, and the first safe verification step. If a fact is unknown, say so plainly instead of inferring it.
