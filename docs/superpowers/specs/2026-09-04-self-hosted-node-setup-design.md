# Self-Hosted Node Setup Skill Design

## Goal

把本项目关于 VLESS + REALITY、住宅代理出口、订阅导入、卡顿排查和 Hysteria2 对比测试的会话经验，提炼为一个可复用的 Codex skill：`self-hosted-node-setup`。

技能面向用户自有或明确获授权的 VPS、代理账号和客户端环境。它指导安全部署、验证、性能判断和回滚，不把某一次会话的地址、凭据、版本结论或出口 IP 当成通用事实。

## Scope

### In scope

- VPS + 单独购买住宅 HTTP/HTTPS/SOCKS5 出口的常见拓扑。
- 住宅 IP 直接绑定 VPS、上游代理、GRE/WireGuard/BGP/端口转发三类交付方式的识别。
- Mihomo 原生 VLESS + REALITY 入站作为基线方案。
- UUID、REALITY X25519 密钥对和 Short ID 的生成、对应关系和凭据隔离。
- Clash/Mihomo 客户端配置、代理组、分流、TUN/系统代理关系和订阅导入。
- 服务状态、端口、出口 IP、DNS、重启恢复和日志的逐层验证。
- 在高 RTT、抖动或丢包环境下，以 Hysteria2 为可选外层做可比 A/B 测试。
- BBR、qdisc、MTU/MSS 等网络实验的临时变更、证据记录和回滚。

### Out of scope

- 未授权访问第三方系统、绕过访问控制或规避服务商条款。
- 代替供应商隧道文档实现未知的 GRE/WireGuard/BGP 路由。
- 将“1TB/30天”显示名称误当作限额和计费实现；这需要独立的面板、统计和到期逻辑。
- 自动停止、覆盖或迁移现有 Nginx、Caddy、Xray、x-ui 等生产服务。

## Skill layout

技能的可发现安装目标为 `~/.agents/skills/self-hosted-node-setup/`。入口保持短小，详细流程按条件放在参考文件中：

```text
self-hosted-node-setup/
├── SKILL.md
└── references/
    ├── intake-and-safety.md
    ├── vless-reality-mihomo.md
    ├── residential-upstream.md
    ├── hysteria2-ab-testing.md
    ├── client-and-subscription.md
    └── diagnosis-and-rollback.md
```

不添加脚本、资产或项目专用配置；只有在未来出现可重复且低风险的机械步骤时，才单独引入脚本。

## Entrypoint responsibilities

`SKILL.md` 仅包含：

1. 触发描述和授权边界。
2. 住宅资源交付方式的路由表。
3. 必须遵守的安全不变量：先只读预检、先备份再改、凭据不进聊天/Git、私钥不进客户端、上游 UDP 未证实时保持关闭、现有服务不抢占端口。
4. 部署阶段顺序和“上一层失败即停止”的验证门槛。
5. 何时读取六个参考文件。
6. 输出格式：当前拓扑、已验证事实、下一条命令、预期结果、失败时的回滚或停止动作。

## Workflow

1. **Intake**：只收集系统/架构、VPS 公网地址、监听端口、住宅协议、认证方式、UDP 能力和套餐限制等非敏感信息。
2. **Topology decision**：区分 VPS 直绑地址、住宅上游代理和隧道/路由交付；信息不完整时停止。
3. **Preflight**：检查时间同步、云防火墙/UFW、现有监听者、磁盘和服务状态；443 被占用时选择空闲端口，不直接终止进程。
4. **Baseline deploy**：安装并校验 Mihomo；生成全新凭据；配置住宅出站、VLESS + REALITY 入站、systemd 和客户端 YAML。
5. **Layered verification**：服务状态 → 上游单独出口 → VLESS 握手 → HTTPS 访问 → 出口 IP/DNS → 重启恢复。每层失败只收集脱敏证据，不叠加猜测性改动。
6. **Optional performance branch**：基线稳定后，才评估 Hysteria2；证书、UDP 443 和回退入口必须先准备好；使用同出口、同目标、同数据量的多轮测试。
7. **Maintenance**：保存不含秘密的说明和受控备份；升级或网络参数变更后复跑验证；凭据泄露时同时轮换相关身份材料。

## Verification contract

部署完成必须有新鲜证据证明：

- 配置解析成功，systemd 服务 active，预期 TCP/UDP 端口正在监听。
- 从 VPS 单独访问住宅代理时，出口 IP 等于购买的住宅 IP，并在多次测试中稳定。
- 客户端能完成 VLESS + REALITY 或 Hysteria2 握手，业务 HTTPS 请求返回有效状态码。
- 出口不是 VPS 机房 IP，DNS 和分流结果符合预期。
- 重启服务或 VPS 后能自动恢复，且已有服务仍可用。
- 性能结论包含 RTT、TLS/握手、TTFB、总耗时、吞吐和最慢样本，而不是只看客户端延迟数字。

## Safety and rollback

- 密码使用交互式输入或受限文件；命令、日志和文档均脱敏。
- 真实配置和私钥只存放在受控位置，服务端私钥永不复制到客户端。
- 修改前保留带时间戳的配置和二进制备份，并记录恢复命令。
- Hysteria2、qdisc、BBR、MTU/MSS 等实验先临时应用；新链路失败立即切回 VLESS。
- 任何无法确认的 UDP、端口转发、证书路径或供应商限制，都标记为未知并停止继续部署。

## Source mapping

- `定位实现方式.md`：Mihomo/Clash VLESS + REALITY 字段含义、住宅代理拓扑和凭据边界。
- `排查订阅问题.md`：会话归档失败本身不改变技能需求；技能应允许从只读预检重新开始。
- `继续排查订阅代理卡顿.md`：9443/443 端口错配、TUN 与系统代理重复接管、IPv6/连接并发、出口核验和分层测速证据。
- `2026-09-04-vless-vps-optimization-session.md`：BBR 与 qdisc 的实测方法、Hysteria2/VLESS A/B 指标、住宅上游不是主要瓶颈以及保留回退节点的决策。
- `docs/2026-09-03-vless-reality-residential-proxy-deployment-task.md`：阶段化部署清单、Mihomo 服务端字段、敏感信息约束和故障定位顺序。

## Acceptance criteria

- `SKILL.md` 的 frontmatter 合法、描述以触发条件为主且可被准确发现。
- 入口不超过必要长度，六个参考文件均能从入口按条件找到。
- 所有示例使用占位符，不包含真实 IP、UUID、密码、私钥、Short ID、订阅 URL 或令牌。
- 文档没有未完成占位项、相互矛盾的端口/协议假设或“UDP 一定更快”等未经测量的结论。
- 用 validator 检查通过，并用至少一个真实压力场景验证未来 agent 会先辨别拓扑、先做备份和分层验证。
