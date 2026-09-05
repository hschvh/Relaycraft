# Relaycraft

Safety-first skills for authorized self-hosted relay and proxy workflows.

Relaycraft turns repeatable deployment and diagnosis practices into discoverable Codex skills. The first skill, `relay`, covers self-hosted VPS nodes, residential upstream proxies, Mihomo/Clash clients, performance comparisons, and recoverable rollback.

## Scope and safety

Use these materials only with VPSs, proxy accounts, and client environments that you own or are explicitly authorized to administer.

- Never put passwords, UUIDs, private keys, Short IDs, subscription URLs, or tokens in chat, Git, logs, or screenshots.
- Preserve existing Nginx, Caddy, Xray, x-ui, and other services. Do not take over an occupied port without explicit direction.
- Treat UDP support, tunnel routing, certificates, and provider limits as unknown until documented or measured.
- Back up before mutation, verify one layer at a time, and stop at the first failed layer.
- Do not claim that Hysteria2, BBR, or another optimization is faster without comparable measurements.

This repository documents authorized workflows. It does not connect to, configure, or monitor a live VPS for you.

## Included skill

### `relay`

The `relay` skill is available at [`skills/relay/SKILL.md`](./skills/relay/SKILL.md). It routes an agent to the smallest relevant reference for the current topology or symptom:

| Reference | Use it when |
| --- | --- |
| [`intake-and-safety.md`](./skills/relay/references/intake-and-safety.md) | Classifying delivery topology and collecting non-sensitive facts |
| [`vless-reality-mihomo.md`](./skills/relay/references/vless-reality-mihomo.md) | Establishing the baseline VLESS + REALITY path |
| [`residential-upstream.md`](./skills/relay/references/residential-upstream.md) | Validating an HTTP/HTTPS/SOCKS5 residential upstream |
| [`hysteria2-ab-testing.md`](./skills/relay/references/hysteria2-ab-testing.md) | Running a fair, optional Hysteria2 comparison after the baseline is stable |
| [`client-and-subscription.md`](./skills/relay/references/client-and-subscription.md) | Configuring Mihomo clients, subscriptions, routing, and exit checks |
| [`diagnosis-and-rollback.md`](./skills/relay/references/diagnosis-and-rollback.md) | Investigating failures and restoring a known-good state |

## Workflow

The skill follows seven phases:

1. Intake only non-sensitive topology facts.
2. Classify direct binding, upstream proxy, or tunnel/routing delivery.
3. Run read-only preflight checks.
4. Deploy and verify a baseline path.
5. Verify service, listener, handshake, HTTPS, exit IP/DNS, and restart recovery in layers.
6. Compare Hysteria2 only after the baseline is stable and the UDP/certificate prerequisites are verified.
7. Maintain backups, evidence, and rollback instructions.

## Install

Clone the repository, then install the skill into the shared agent skill directory:

```bash
git clone https://github.com/hschvh/Relaycraft.git
cd Relaycraft
mkdir -p ~/.agents/skills
cp -a ./skills/relay ~/.agents/skills/relay
```

If `~/.agents/skills/relay` already exists, back it up before replacing it. The skill is documentation and routing guidance; it does not install Mihomo or alter system services by itself.

## Use

Ask your agent for an authorized self-hosted node task and provide only non-sensitive facts such as the VPS platform, architecture, candidate listeners, residential protocol, authentication category, and known UDP support. Enter secrets interactively on the authorized host or through a restricted file when the reference requires them.

For a slow client, preserve the working baseline and collect the same-exit, same-target, same-payload RTT, TLS/handshake, TTFB, total time, throughput, and worst sample across at least five repetitions before changing protocol or kernel settings.

## Validate

From a checkout containing the skill, run the bundled skill validator when it is available:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/relay
```

Expected output:

```text
Skill is valid!
```

The source tree should contain one entrypoint, one UI metadata file, and six Markdown references:

```text
skills/relay/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── client-and-subscription.md
    ├── diagnosis-and-rollback.md
    ├── hysteria2-ab-testing.md
    ├── intake-and-safety.md
    ├── residential-upstream.md
    └── vless-reality-mihomo.md
```

## Contributing

Keep examples placeholder-only, preserve the authorized-use boundary, and add a verification or rollback gate when documenting a new workflow. Do not commit live credentials or provider-specific secrets.
