---
name: self-hosted-node-setup
description: Use when setting up or diagnosing an authorized self-hosted proxy node on a VPS, especially VLESS/REALITY or Hysteria2 with a residential HTTP/SOCKS5 upstream, Mihomo/Clash clients, subscriptions, or high-latency/packet-loss symptoms.
---

# Self-Hosted Node Setup

Use this skill only for user-owned or explicitly authorized VPS, proxy-account, and client environments.

Follow the seven phases in order: intake, topology decision, read-only preflight, baseline deploy, layered verification, optional Hysteria2 comparison, and maintenance/rollback.

Hard invariants:

- Do not connect to or modify a real VPS unless the user has authorized that environment.
- Do not ask for, store, or paste passwords, UUIDs, private keys, Short IDs, subscription URLs, or tokens into chat or git.
- Do not stop, overwrite, or seize an existing Nginx, Caddy, Xray, x-ui, or similar service.
- Keep UDP disabled until provider support or end-to-end behavior is verified.
- If a lower layer fails, stop there. Collect redacted evidence and do not stack extra changes on top.
- Back up before mutation and keep a rollback command beside every change.

Read only the references that match the current topology or symptom:

- [intake-and-safety](references/intake-and-safety.md) when you need the minimum non-sensitive intake or need to classify the topology.
- [vless-reality-mihomo](references/vless-reality-mihomo.md) for the baseline Mihomo VLESS + REALITY deployment path.
- [residential-upstream](references/residential-upstream.md) when the residential resource is an upstream HTTP/HTTPS/SOCKS5 proxy.
- [hysteria2-ab-testing](references/hysteria2-ab-testing.md) only after the baseline path is stable and you want a fair Hysteria2 comparison.
- [client-and-subscription](references/client-and-subscription.md) for Mihomo client setup, subscription import, and exit-IP checks.
- [diagnosis-and-rollback](references/diagnosis-and-rollback.md) when a layer fails, the exit is wrong, or you need to revert safely.

Report the current topology, the non-sensitive facts verified so far, the next safe check, and the expected stop or rollback action if that check fails.
