# Intake and Safety

Use this reference before any deployment or configuration mutation. The workflow is limited to a VPS, residential proxy account, and client environment that the operator owns or is explicitly authorized to administer.

## Classify the topology

Choose exactly one branch from the facts supplied by the provider:

1. **Direct address binding:** the residential address is routed to the VPS through a documented provider feature. A host/port/login alone does not prove this branch.
2. **Residential upstream proxy:** the provider supplied an HTTP, HTTPS, or SOCKS5 endpoint. Mihomo uses it as an outbound hop; the client still connects to the VPS.
3. **Tunnel or routed delivery:** the provider supplied a GRE, WireGuard, BGP, or port-forwarding design with routing instructions. Do not invent implementation details; use the provider's documented procedure.

If the provider only supplied `host`, `port`, `username`, and `password`, classify it as an upstream proxy until documented evidence proves otherwise.

## Minimum non-sensitive intake

| Fact | Safe form to request | Why it matters |
| --- | --- | --- |
| Authorization | Confirmation that the VPS, proxy account, and clients are in scope | Establishes the safety boundary |
| VPS platform | Distribution/version and CPU architecture | Selects package and binary |
| VPS endpoint | `<VPS_PUBLIC_IP>` or `<VPS_HOSTNAME>` | Identifies the client server field |
| Existing listeners | Service names and candidate ports only | Prevents port takeover |
| Residential delivery | Direct bind, HTTP/HTTPS, SOCKS5, or tunnel/routing | Selects the reference path |
| Authentication form | Password, token, IP allowlist, or none; never the value | Chooses safe input handling |
| UDP capability | `unknown` until provider and end-to-end checks confirm it | Avoids false Hysteria2 assumptions |
| Certificate and limits | Certificate availability, bandwidth/expiry policy, and documented restrictions | Gates optional protocols |

Request secrets through an interactive prompt or a restricted file on the authorized host. Never ask the user to paste passwords, UUIDs, private keys, Short IDs, subscription URLs, or tokens into chat, logs, Markdown, or Git. Examples must use placeholders such as `<UPSTREAM_PASSWORD>`.

## Hard stop conditions

Stop and request clarification when:

- authorization is absent or ambiguous;
- topology, port ownership, certificate location, or UDP capability is unknown and the next action could mutate or expose a service;
- a requested change would stop, overwrite, or bind an occupied Nginx, Caddy, Xray, x-ui, or other service;
- a command would print a secret or write one into the repository;
- the user asks to bypass provider limits, access a third-party system, or conceal activity.

Before any mutation, take a timestamped backup and write the restoration command beside it. Run read-only checks first; a failed layer is a gate, not an invitation to stack another change.

## Routing rule

For a residential upstream proxy, the client `server` is `<VPS_PUBLIC_IP>` (or the VPS hostname). The residential host belongs in the VPS-side outbound proxy definition. Do not put the residential endpoint in the client as if it were the VLESS server.

## Required progress record

Report the current topology, facts verified, facts still unknown, next safe command, expected result, and the stop or rollback action if it fails. Keep command output redacted and record no live credentials.
