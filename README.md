# Relaycraft

面向已授权自建中继和代理工作流的安全优先技能集合。\
Safety-first skills for authorized self-hosted relay and proxy workflows.

Relaycraft 将可重复的部署、验证、诊断和回滚经验整理为可发现的 Codex skills。第一个技能 `relay` 覆盖自建 VPS 节点、住宅上游代理、Mihomo/Clash 客户端、性能对比和可恢复回滚。\
Relaycraft turns repeatable deployment, verification, diagnosis, and rollback practices into discoverable Codex skills. The first skill, `relay`, covers self-hosted VPS nodes, residential upstream proxies, Mihomo/Clash clients, performance comparisons, and recoverable rollback.

[中文](#中文) | [English](#english)

## 中文

[切换到 English](#english)

### 范围与安全

本仓库仅适用于你拥有或明确获授权管理的 VPS、代理账号和客户端环境。

- 不要把密码、UUID、私钥、Short ID、订阅 URL 或令牌写入聊天、Git、日志或截图。
- 保留现有 Nginx、Caddy、Xray、x-ui 和其他服务；未经明确指示，不要接管已占用端口。
- UDP 能力、隧道路由、证书和供应商限制在有文档或实测证据前都视为未知。
- 变更前先备份，逐层验证，任一层失败就停止。
- 没有同出口、同目标、同 payload 的可比数据，不要声称 Hysteria2、BBR 或其他优化更快。

本仓库只提供授权工作流文档和路由指导，不会替你连接、配置或监控真实 VPS。

### 包含的技能

入口文件是 [`skills/relay/SKILL.md`](./skills/relay/SKILL.md)。它会根据当前拓扑或症状路由到最小必要的参考文档：

| 参考文档 | 适用场景 |
| --- | --- |
| [`intake-and-safety.md`](./skills/relay/references/intake-and-safety.md) | 判定交付拓扑并收集非敏感事实 |
| [`vless-reality-mihomo.md`](./skills/relay/references/vless-reality-mihomo.md) | 建立 VLESS + REALITY 基线链路 |
| [`residential-upstream.md`](./skills/relay/references/residential-upstream.md) | 验证 HTTP/HTTPS/SOCKS5 住宅上游 |
| [`hysteria2-ab-testing.md`](./skills/relay/references/hysteria2-ab-testing.md) | 基线稳定后进行可比的 Hysteria2 对照测试 |
| [`client-and-subscription.md`](./skills/relay/references/client-and-subscription.md) | 配置 Mihomo 客户端、订阅、路由和出口检查 |
| [`diagnosis-and-rollback.md`](./skills/relay/references/diagnosis-and-rollback.md) | 排查故障并恢复已知良好状态 |

### 工作流

`relay` 按七个阶段工作：

1. 只收集非敏感拓扑事实。
2. 区分直接绑定、上游代理和隧道/路由交付。
3. 执行只读预检。
4. 部署并验证基线链路。
5. 逐层验证服务、监听、握手、HTTPS、出口 IP/DNS 和重启恢复。
6. 只有基线稳定且 UDP/证书前置条件已验证后，才比较 Hysteria2。
7. 维护备份、证据和回滚说明。

### 安装

克隆仓库，然后把技能安装到共享 agent skill 目录：

```bash
git clone https://github.com/hschvh/Relaycraft.git
cd Relaycraft
mkdir -p ~/.agents/skills
cp -a ./skills/relay ~/.agents/skills/relay
```

如果 `~/.agents/skills/relay` 已存在，请先备份再替换。该技能只提供文档和路由指导，不会自行安装 Mihomo 或修改系统服务。

### 使用

向 agent 提出已授权的自建节点任务，只提供 VPS 平台、架构、候选监听、住宅协议、认证类别和已知 UDP 能力等非敏感事实。参考文档要求输入秘密时，应在授权主机上交互输入，或使用受限文件。

遇到客户端变慢时，先保留工作基线，在修改协议或内核参数前，用同一出口、同一目标、同一 payload，至少重复五次采集 RTT、TLS/握手、TTFB、总耗时、吞吐和最慢样本。

### 验证

在包含该技能的 checkout 中，使用可用的 bundled skill validator：

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/relay
```

预期输出：

```text
Skill is valid!
```

源目录应包含一个入口、一个 UI 元数据文件和六个 Markdown references：

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

### 贡献

示例必须只使用占位符，保留授权使用边界；新增工作流时必须补充验证或回滚门槛。不要提交真实凭据或供应商专属秘密。

## English

[Switch to 中文](#中文)

### Scope and safety

Use these materials only with VPSs, proxy accounts, and client environments that you own or are explicitly authorized to administer.

- Never put passwords, UUIDs, private keys, Short IDs, subscription URLs, or tokens in chat, Git, logs, or screenshots.
- Preserve existing Nginx, Caddy, Xray, x-ui, and other services. Do not take over an occupied port without explicit direction.
- Treat UDP support, tunnel routing, certificates, and provider limits as unknown until documented or measured.
- Back up before mutation, verify one layer at a time, and stop at the first failed layer.
- Do not claim that Hysteria2, BBR, or another optimization is faster without comparable measurements.

This repository documents authorized workflows. It does not connect to, configure, or monitor a live VPS for you.

### Included skill

The `relay` skill is available at [`skills/relay/SKILL.md`](./skills/relay/SKILL.md). It routes an agent to the smallest relevant reference for the current topology or symptom:

| Reference | Use it when |
| --- | --- |
| [`intake-and-safety.md`](./skills/relay/references/intake-and-safety.md) | Classifying delivery topology and collecting non-sensitive facts |
| [`vless-reality-mihomo.md`](./skills/relay/references/vless-reality-mihomo.md) | Establishing the baseline VLESS + REALITY path |
| [`residential-upstream.md`](./skills/relay/references/residential-upstream.md) | Validating an HTTP/HTTPS/SOCKS5 residential upstream |
| [`hysteria2-ab-testing.md`](./skills/relay/references/hysteria2-ab-testing.md) | Running a fair, optional Hysteria2 comparison after the baseline is stable |
| [`client-and-subscription.md`](./skills/relay/references/client-and-subscription.md) | Configuring Mihomo clients, subscriptions, routing, and exit checks |
| [`diagnosis-and-rollback.md`](./skills/relay/references/diagnosis-and-rollback.md) | Investigating failures and restoring a known-good state |

### Workflow

The skill follows seven phases:

1. Intake only non-sensitive topology facts.
2. Classify direct binding, upstream proxy, or tunnel/routing delivery.
3. Run read-only preflight checks.
4. Deploy and verify a baseline path.
5. Verify service, listener, handshake, HTTPS, exit IP/DNS, and restart recovery in layers.
6. Compare Hysteria2 only after the baseline is stable and the UDP/certificate prerequisites are verified.
7. Maintain backups, evidence, and rollback instructions.

### Install

Clone the repository, then install the skill into the shared agent skill directory:

```bash
git clone https://github.com/hschvh/Relaycraft.git
cd Relaycraft
mkdir -p ~/.agents/skills
cp -a ./skills/relay ~/.agents/skills/relay
```

If `~/.agents/skills/relay` already exists, back it up before replacing it. The skill is documentation and routing guidance; it does not install Mihomo or alter system services by itself.

### Use

Ask your agent for an authorized self-hosted node task and provide only non-sensitive facts such as the VPS platform, architecture, candidate listeners, residential protocol, authentication category, and known UDP support. Enter secrets interactively on the authorized host or through a restricted file when the reference requires them.

For a slow client, preserve the working baseline and collect the same-exit, same-target, same-payload RTT, TLS/handshake, TTFB, total time, throughput, and worst sample across at least five repetitions before changing protocol or kernel settings.

### Validate

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

### Contributing

Keep examples placeholder-only, preserve the authorized-use boundary, and add a verification or rollback gate when documenting a new workflow. Do not commit live credentials or provider-specific secrets.
