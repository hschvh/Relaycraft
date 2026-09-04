# Self-Hosted Node Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create and validate a discoverable `self-hosted-node-setup` skill that guides authorized VPS + residential-proxy node deployment, optional Hysteria2 comparison, and evidence-based rollback.

**Architecture:** Keep a concise `SKILL.md` as the routing and safety entrypoint. Put protocol-specific procedures, command patterns, test gates, and rollback details in six focused Markdown references under `references/`; keep the source in the repository and copy the completed directory into `~/.agents/skills/` so Codex can discover it.

**Tech Stack:** Markdown, YAML frontmatter, Python `quick_validate.py` from the bundled skill-creator package, POSIX shell commands shown as documented templates, Git.

## Global Constraints

- Scope is authorized use of a VPS, residential proxy account, and client environment.
- Preserve existing Nginx, Caddy, Xray, x-ui, and other services; never seize an occupied port or stop a service without explicit user direction.
- Generate fresh UUID, REALITY keypair, and Short ID for each deployment; never reuse source-document values.
- Never place real IPs, credentials, UUIDs, private keys, Short IDs, subscription URLs, or tokens in the skill or repository.
- Treat UDP support, tunnel routing, certificate paths, and provider limits as unknown until verified.
- Require backup before mutation, layered verification, and an explicit rollback path.
- Do not claim Hysteria2 is faster without same-exit, same-target, same-payload measurements.

---

### Task 1: Initialize the skill directory

**Files:**
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/SKILL.md`
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/`
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/agents/openai.yaml`

**Interfaces:**
- Consumes: the approved design in `docs/superpowers/specs/2026-09-04-self-hosted-node-setup-design.md`.
- Produces: a discoverable skill folder with valid frontmatter and six reference slots.

- [ ] **Step 1: Run the bundled initializer**

Run:

```bash
mkdir -p /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills
python3 /Users/tanbin/.codex/skills/.system/skill-creator/scripts/init_skill.py \
  self-hosted-node-setup \
  --path /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills \
  --resources references
```

Expected: the command creates `SKILL.md`, `agents/openai.yaml`, and `references/` under `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/`.

- [ ] **Step 2: Confirm the generated paths and remove scaffold-only content**

Run:

```bash
find /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup -maxdepth 2 -type f -print | sort
```

Expected: only the generated entrypoint, UI metadata, and reference directory are present before content is written.

- [ ] **Step 3: Commit the skeleton**

```bash
git add skills/self-hosted-node-setup docs/superpowers/plans/2026-09-04-self-hosted-node-setup.md
git commit -m "docs: plan self-hosted node setup skill"
```

### Task 2: Write the routing and safety entrypoint

**Files:**
- Modify: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/SKILL.md`
- Modify: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/agents/openai.yaml`

**Interfaces:**
- Consumes: the six reference filenames from Task 1.
- Produces: a short, trigger-focused entrypoint that routes an agent to exactly the references needed for the current topology or symptom.

- [ ] **Step 1: Replace the scaffold with the approved frontmatter and routing contract**

Write a `SKILL.md` whose frontmatter contains:

```yaml
---
name: self-hosted-node-setup
description: Use when setting up or diagnosing an authorized self-hosted proxy node on a VPS, especially VLESS/REALITY or Hysteria2 with a residential HTTP/SOCKS5 upstream, Mihomo/Clash clients, subscriptions, or high-latency/packet-loss symptoms.
---
```

The body must state the authorized-use boundary, the seven workflow phases, the hard safety invariants, the “previous layer failed means stop” rule, and links to each reference with a condition for reading it.

- [ ] **Step 2: Set UI metadata without changing implicit discovery**

Set `agents/openai.yaml` display text to identify the skill as “Self-hosted proxy node setup” and keep implicit invocation enabled. Do not add a policy that requires explicit invocation.

- [ ] **Step 3: Check entrypoint size and links**

Run:

```bash
wc -w /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/SKILL.md
rg -n 'intake-and-safety|vless-reality-mihomo|residential-upstream|hysteria2-ab-testing|client-and-subscription|diagnosis-and-rollback' /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/SKILL.md
```

Expected: the entrypoint is concise enough for routine loading and contains one discoverable link for each reference.

- [ ] **Step 4: Commit the entrypoint**

```bash
git add skills/self-hosted-node-setup/SKILL.md skills/self-hosted-node-setup/agents/openai.yaml
git commit -m "feat: add self-hosted node skill entrypoint"
```

### Task 3: Add intake, deployment, and residential-upstream references

**Files:**
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/intake-and-safety.md`
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/vless-reality-mihomo.md`
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/residential-upstream.md`

**Interfaces:**
- Consumes: topology and safety rules from Task 2.
- Produces: actionable protocol/deployment references using placeholders and redacted-output examples only.

- [ ] **Step 1: Write intake and safety rules**

Include the three topology branches, the minimum non-sensitive intake table, interactive secret input guidance, explicit stop conditions, and the rule that the client `server` is the VPS address when the residential resource is an upstream proxy.

- [ ] **Step 2: Write the Mihomo VLESS + REALITY procedure**

Include read-only preflight commands, architecture-aware binary verification, fresh credential generation, placeholder server/client YAML, systemd setup, port selection when 443 is occupied, and syntax/listener checks. State that the server keeps the private key and the client receives only the public key.

- [ ] **Step 3: Write residential upstream variants**

Include placeholder Mihomo structures for HTTP/HTTPS and SOCKS5, an interactive `curl` exit-IP test, authentication variants, static-exit checks, and the default `udp: false` rule until provider support is documented or measured. Explain that `udp: true` on the client does not convert ordinary TCP web requests into UDP.

- [ ] **Step 4: Scan these references for secrets and contradictions**

Run:

```bash
rg -n '182\\.255|204\\.1|144\\.202|uuid: [0-9a-f-]{8}|private-key: [A-Za-z0-9_-]{20,}|password: [^<\\" ]+|short-id: [0-9a-f]{8,}|https?://[^ >]+subscribe' \
  /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references || true
```

Expected: no real credential-like values or project-specific addresses are found; all examples use placeholders.

- [ ] **Step 5: Commit the three references**

```bash
git add skills/self-hosted-node-setup/references/intake-and-safety.md skills/self-hosted-node-setup/references/vless-reality-mihomo.md skills/self-hosted-node-setup/references/residential-upstream.md
git commit -m "feat: document vless reality and residential upstream setup"
```

### Task 4: Add client, Hysteria2, and diagnosis references

**Files:**
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/hysteria2-ab-testing.md`
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/client-and-subscription.md`
- Create: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/diagnosis-and-rollback.md`

**Interfaces:**
- Consumes: deployment fields and safety invariants from Task 3.
- Produces: optional performance branch, client workflow, symptom matrix, and recoverable experiment procedures.

- [ ] **Step 1: Write the Hysteria2 A/B reference**

Include certificate and UDP 443 preconditions, VLESS fallback preservation, a test table capturing RTT/TLS/TTFB/total/throughput/worst sample, five-or-more repeated samples, and a rule that BBR/Brutal conclusions must follow the actual server configuration. Include temporary qdisc/BBR experiments with immediate restoration commands.

- [ ] **Step 2: Write client and subscription guidance**

Include Mihomo proxy/group/rule placeholders, the relationship between TUN and system proxy, IPv6 and `tcp-concurrent` as measured toggles rather than universal fixes, offline config validation, permissions, import steps, and exit-IP verification.

- [ ] **Step 3: Write diagnosis and rollback guidance**

Include a symptom-to-first-check table for port failure, REALITY handshake failure, upstream authentication, wrong exit, partial site failure, UDP failure, and slowness. Include redacted `ss`, `journalctl`, `curl`, and proxy health probes, backup naming, service/config rollback, and the rule to change one variable at a time.

- [ ] **Step 4: Check cross-reference discoverability**

Run:

```bash
for ref in /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/*.md; do
  printf '%s: ' "$ref"
  rg -l 'stop|验证|rollback|回滚|placeholder|占位' "$ref" >/dev/null && echo ok || echo missing-required-concept
done
```

Expected: every reference contains an observable stop/verification/rollback concept appropriate to its mode.

- [ ] **Step 5: Commit the three references**

```bash
git add skills/self-hosted-node-setup/references/hysteria2-ab-testing.md skills/self-hosted-node-setup/references/client-and-subscription.md skills/self-hosted-node-setup/references/diagnosis-and-rollback.md
git commit -m "feat: document hysteria2 testing and rollback"
```

### Task 5: Validate structure, privacy, and internal consistency

**Files:**
- Validate: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/`
- Inspect: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/docs/superpowers/specs/2026-09-04-self-hosted-node-setup-design.md`

**Interfaces:**
- Consumes: the completed skill tree from Tasks 2–4.
- Produces: validator output, a clean diff-free content audit, and a list of any corrected contradictions.

- [ ] **Step 1: Run the bundled validator**

Run:

```bash
python3 /Users/tanbin/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup
```

Expected: exit code 0 with no frontmatter, naming, or scaffold-placeholder errors.

- [ ] **Step 2: Run the Markdown hygiene checks**

Run:

```bash
rg -n 'TODO|TBD|FIXME|<REDACTED>|your-real|replace-me|password: [^<\" ]+|private-key: [^<\" ]+' \
  /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup || true
git diff --check
```

Expected: no unfinished placeholders or secret-like literals; `git diff --check` reports no whitespace errors.

- [ ] **Step 3: Audit design coverage**

Check each design acceptance criterion against the actual tree: valid frontmatter, six routed references, placeholder-only examples, no contradictory port/UDP assumptions, and explicit rollback gates. Correct omissions in the relevant file before proceeding.

- [ ] **Step 4: Commit validation corrections**

```bash
git add skills/self-hosted-node-setup
git commit -m "chore: validate self-hosted node skill content"
```

### Task 6: Run a realistic pressure scenario and package the result

**Files:**
- Read: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/SKILL.md`
- Read: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup/references/*.md`
- Inspect: `/Users/tanbin/Documents/ChatGPT/CPA-deploy/docs/superpowers/specs/2026-09-04-self-hosted-node-setup-design.md`

**Interfaces:**
- Consumes: validated skill instructions and the source-derived acceptance criteria.
- Produces: an evidence-backed pressure-scenario result showing that the skill routes safely under ambiguous topology and slowness symptoms.

- [ ] **Step 1: Use an isolated scratch prompt without mutating a live VPS**

Evaluate this scenario against the skill: “I bought a static residential IP, but the provider only sent host, port, username, and password. My VPS already has Nginx on 443. Build the node quickly; the client says ChatGPT is reachable but slow.”

The expected behavior is to classify the residential resource as an upstream proxy, keep Nginx, choose an unused TCP port, request only non-sensitive facts, test the residential exit independently, avoid claiming UDP support, and defer Hysteria2 until baseline verification and comparable measurements exist.

- [ ] **Step 2: Record the observable checks**

Confirm the response contains: topology classification, hard stop or missing-input checks, backup-before-mutation, layered verification, redacted secret handling, no direct 443 takeover, and A/B metrics beyond a single latency number.

- [ ] **Step 3: Re-run final validator and report evidence**

Run:

```bash
python3 /Users/tanbin/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup
find /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup -maxdepth 2 -type f -print | sort
```

Expected: validator exit code 0 and exactly one `SKILL.md`, one `agents/openai.yaml`, and six Markdown references.

- [ ] **Step 4: Deliver the installation path and evidence**

Install the validated source into the discoverable directory:

```bash
mkdir -p /Users/tanbin/.agents/skills
cp -a /Users/tanbin/Documents/ChatGPT/CPA-deploy/skills/self-hosted-node-setup /Users/tanbin/.agents/skills/
```

Then report both absolute paths, validator output, pressure-scenario findings, and the design/plan commit IDs. Do not claim live VPS deployment or network performance; this skill package only documents the authorized workflow.
