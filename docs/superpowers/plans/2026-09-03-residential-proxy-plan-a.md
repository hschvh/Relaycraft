# 固定住宅代理方案 A 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在保持固定住宅出口 IP 不变的前提下，通过 VPS 直连/住宅代理四象限测速确定 Cloudflare 下行瓶颈，并形成可提交给供应商的脱敏证据。

**Architecture:** 所有探针先在当前 VPS 本机执行，分别测试 VPS 直连和读取现有 Mihomo SOCKS5 出站后的代理访问。住宅用户名和密码只通过标准输入传给 curl，不写入命令行、报告或仓库；本计划不修改 Mihomo、Nginx、Xray、3x-ui、防火墙或客户端配置。

**Tech Stack:** Ubuntu 24.04、systemd、Mihomo、curl、Python 3、Bash、Cloudflare Speedtest endpoint、Google CDN

## Global Constraints

- 必须保留当前固定住宅出口 IP `204.1.x.x`；不允许业务流量意外变为 VPS 机房出口。
- 所有境外流量继续使用当前住宅出口；本计划不改变客户端分流规则。
- 当前 VLESS + REALITY/TCP 443 保持运行，且必须在测试前后均可用。
- 不停止或抢占 Nginx、Xray、3x-ui 和其他生产服务。
- 住宅代理用户名、密码、VLESS UUID、REALITY 私钥及完整出口 IP 不进入报告、仓库或聊天记录。
- 方案 A 的第一批任务只读；在基线结论明确前不切换供应商网关。

## 回滚边界

本计划不修改任何生产配置或服务状态，因此回滚只需要退出当前 SSH shell，使临时函数和变量消失。报告文件保留为证据；不得为了“恢复”而重启服务、覆盖配置或切换网关。

---

### Task 1: VPS 只读预检

**Files:**
- Read: `/etc/mihomo-residential/config.yaml`
- Read: `/etc/systemd/system/mihomo-residential.service`
- Create: `/root/residential-benchmark-20260903.txt`

**Interfaces:**
- Consumes: 当前 VPS 的 Mihomo JSON 配置和 systemd 服务状态。
- Produces: 不含凭据的服务、监听、出站类型和上游地址脱敏摘要。

- [x] **Step 1: 建立权限受限的报告文件**

Run on the current VPS:

```bash
umask 077
REPORT=/root/residential-benchmark-20260903.txt
: > "$REPORT"
chmod 600 "$REPORT"
```

Expected: `ls -l "$REPORT"` 显示权限为 `-rw-------`。

- [x] **Step 2: 验证现有服务与 TCP 443 监听**

```bash
{
  printf 'service='; systemctl is-active mihomo-residential.service
  printf 'enabled='; systemctl is-enabled mihomo-residential.service
  ss -lntp | awk '$4 ~ /:443$/ {print "tcp443=" $4 " process=" $NF}'
} | tee -a "$REPORT"
```

Expected: 服务为 `active`，且 TCP 443 存在当前 VLESS/REALITY 入口监听。若服务名称或监听归属不符，停止本计划并先确认实际入口服务。

- [x] **Step 3: 验证配置格式并只输出脱敏上游信息**

```bash
CFG=/etc/mihomo-residential/config.yaml
sudo python3 - "$CFG" <<'PY' | tee -a "$REPORT"
import ipaddress
import json
import sys
from pathlib import Path

data = json.loads(Path(sys.argv[1]).read_text(encoding="utf-8"))
proxy = next(p for p in data["proxies"] if p["name"] == "residential-out")
assert proxy["type"] == "socks5", proxy["type"]
server = str(proxy["server"])
try:
    ip = ipaddress.ip_address(server)
    if ip.version == 4:
        parts = server.split(".")
        server = f"{parts[0]}.{parts[1]}.x.x"
    else:
        server = "ipv6-redacted"
except ValueError:
    labels = server.split(".")
    server = "*." + ".".join(labels[-2:]) if len(labels) >= 2 else "hostname-redacted"

print("config_format=json")
print("outbound_name=residential-out")
print("outbound_type=socks5")
print(f"upstream={server}:{proxy['port']}")
print(f"udp={bool(proxy.get('udp', False))}")
print("credentials_present=" + str(bool(proxy.get("username")) and bool(proxy.get("password"))))
PY
```

Expected: `outbound_type=socks5`、`credentials_present=True`，输出中不出现完整服务器 IP、用户名或密码。

- [x] **Step 4: 记录当前二进制配置检查结果**

```bash
sudo /usr/local/bin/mihomo-residential \
  -t -d /var/lib/mihomo-residential \
  -f "$CFG" 2>&1 | tail -20 | tee -a "$REPORT"
```

Expected: 结尾包含配置检查成功，且没有解析错误。

---

### Task 2: 建立无凭据泄漏的测速反馈环

**Files:**
- Read: `/etc/mihomo-residential/config.yaml`
- Modify: `/root/residential-benchmark-20260903.txt`

**Interfaces:**
- Consumes: Task 1 验证过的 `residential-out` SOCKS5 配置。
- Produces: 当前 SSH shell 中的 `emit_proxy_config`、`bench_direct` 和 `bench_proxy` 函数。

- [x] **Step 1: 定义只向 curl 标准输入发送凭据的函数**

```bash
emit_proxy_config() {
  sudo python3 - "$CFG" <<'PY'
import json
import sys
from pathlib import Path

data = json.loads(Path(sys.argv[1]).read_text(encoding="utf-8"))
proxy = next(p for p in data["proxies"] if p["name"] == "residential-out")
assert proxy["type"] == "socks5"
endpoint = f"socks5h://{proxy['server']}:{proxy['port']}"
credentials = f"{proxy['username']}:{proxy['password']}"
print("proxy = " + json.dumps(endpoint))
print("proxy-user = " + json.dumps(credentials))
PY
}
```

Expected: `type emit_proxy_config` 显示 shell function。不要直接运行并打印它的输出。

- [x] **Step 2: 定义统一的直连测速函数**

```bash
bench_direct() {
  label="$1"
  url="$2"
  range="$3"
  run="$4"
  if [ "$range" = none ]; then
    curl --http1.1 --max-time 20 --noproxy '*' -sS -o /dev/null \
      -w "path=direct target=$label run=$run code=%{http_code} bytes=%{size_download} tls=%{time_appconnect}s ttfb=%{time_starttransfer}s total=%{time_total}s speed_Bps=%{speed_download} error=%{errormsg}\\n" \
      "$url"
  else
    curl --http1.1 --max-time 20 --noproxy '*' -r "$range" -sS -o /dev/null \
      -w "path=direct target=$label run=$run code=%{http_code} bytes=%{size_download} tls=%{time_appconnect}s ttfb=%{time_starttransfer}s total=%{time_total}s speed_Bps=%{speed_download} error=%{errormsg}\\n" \
      "$url"
  fi
}
```

Expected: `type bench_direct` 显示 shell function。

- [x] **Step 3: 定义统一的住宅代理测速函数**

```bash
bench_proxy() {
  label="$1"
  url="$2"
  range="$3"
  run="$4"
  if [ "$range" = none ]; then
    emit_proxy_config | curl --config - --http1.1 --max-time 20 -sS -o /dev/null \
      -w "path=residential target=$label run=$run code=%{http_code} bytes=%{size_download} tls=%{time_appconnect}s ttfb=%{time_starttransfer}s total=%{time_total}s speed_Bps=%{speed_download} error=%{errormsg}\\n" \
      "$url"
  else
    emit_proxy_config | curl --config - --http1.1 --max-time 20 -r "$range" -sS -o /dev/null \
      -w "path=residential target=$label run=$run code=%{http_code} bytes=%{size_download} tls=%{time_appconnect}s ttfb=%{time_starttransfer}s total=%{time_total}s speed_Bps=%{speed_download} error=%{errormsg}\\n" \
      "$url"
  fi
}
```

Expected: `type bench_proxy` 显示 shell function。`ps` 输出中不得出现住宅用户名或密码。

- [x] **Step 4: 用 100 KB Cloudflare 下载建立红能力测试**

```bash
bench_direct cloudflare \
  'https://speed.cloudflare.com/__down?bytes=100000' none 0 | tee -a "$REPORT"
bench_proxy cloudflare \
  'https://speed.cloudflare.com/__down?bytes=100000' none 0 | tee -a "$REPORT"
```

Expected: 两条记录均下载 `100000` bytes；若住宅代理结果接近 Mac 端已观测的 `16–39 KB/s`，该命令能够稳定捕获当前慢速症状。

---

### Task 3: 执行 VPS 四象限基线

**Files:**
- Modify: `/root/residential-benchmark-20260903.txt`

**Interfaces:**
- Consumes: Task 2 的三个 shell functions。
- Produces: Cloudflare、Google CDN 和 ChatGPT 的直连/住宅代理五次样本。

- [x] **Step 1: 测量直连 Cloudflare 五次**

```bash
for run in 1 2 3 4 5; do
  bench_direct cloudflare \
    'https://speed.cloudflare.com/__down?bytes=100000' none "$run"
done | tee -a "$REPORT"
```

Expected: 五条 `path=direct target=cloudflare` 记录，无 `code=000`。

- [x] **Step 2: 测量住宅代理 Cloudflare 五次**

```bash
for run in 1 2 3 4 5; do
  bench_proxy cloudflare \
    'https://speed.cloudflare.com/__down?bytes=100000' none "$run"
done | tee -a "$REPORT"
```

Expected: 五条 `path=residential target=cloudflare` 记录。任何超时保留在报告中，不重跑覆盖。

- [x] **Step 3: 测量直连 Google CDN 五次**

```bash
for run in 1 2 3 4 5; do
  bench_direct google \
    'https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb' \
    0-99999 "$run"
done | tee -a "$REPORT"
```

Expected: 五条 `path=direct target=google` 记录，每次状态码为 `206`、下载 `100000` bytes。

- [x] **Step 4: 测量住宅代理 Google CDN 五次**

```bash
for run in 1 2 3 4 5; do
  bench_proxy google \
    'https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb' \
    0-99999 "$run"
done | tee -a "$REPORT"
```

Expected: 五条 `path=residential target=google` 记录，每次状态码为 `206`、下载 `100000` bytes。

- [x] **Step 5: 测量直连与住宅代理 ChatGPT 五次**

```bash
for path in direct residential; do
  for run in 1 2 3 4 5; do
    if [ "$path" = direct ]; then
      bench_direct chatgpt 'https://chatgpt.com/' none "$run"
    else
      bench_proxy chatgpt 'https://chatgpt.com/' none "$run"
    fi
  done
done | tee -a "$REPORT"
```

Expected: curl 可能收到 Cloudflare 对非浏览器请求的 `403`，但不得出现 `code=000`；诊断使用 TLS、首字节和总耗时，不把 `403` 误判为链路失败。

---

### Task 4: 验证固定出口并生成统计摘要

**Files:**
- Modify: `/root/residential-benchmark-20260903.txt`

**Interfaces:**
- Consumes: Task 3 的原始样本与当前住宅 SOCKS5 配置。
- Produces: 脱敏出口确认和按 path/target 聚合的平均速度与平均耗时。

- [x] **Step 1: 获取并脱敏输出直连与住宅出口**

```bash
DIRECT_EXIT=$(curl --max-time 10 --noproxy '*' -sS https://api.ipify.org)
RESIDENTIAL_EXIT=$(emit_proxy_config | curl --config - --max-time 10 -sS https://api.ipify.org)
python3 - "$DIRECT_EXIT" "$RESIDENTIAL_EXIT" <<'PY' | tee -a "$REPORT"
import sys

def mask(value):
    parts = value.strip().split(".")
    return ".".join(parts[:2] + ["x", "x"]) if len(parts) == 4 else "redacted"

print("direct_exit=" + mask(sys.argv[1]))
print("residential_exit=" + mask(sys.argv[2]))
print("exits_differ=" + str(sys.argv[1] != sys.argv[2]))
PY
```

Expected: `residential_exit=204.1.x.x`、`exits_differ=True`。若不匹配，停止并检查出站绑定，不能继续供应商调整。

- [x] **Step 2: 汇总直连与住宅路径指标**

```bash
python3 - "$REPORT" <<'PY' | tee -a "$REPORT"
import re
import statistics
import sys
from collections import defaultdict
from pathlib import Path

groups = defaultdict(lambda: {"speed": [], "ttfb": [], "total": [], "errors": 0})
pattern = re.compile(
    r"path=(direct|residential) target=(cloudflare|google|chatgpt) run=([0-9]+).*?"
    r"ttfb=([0-9.]+)s total=([0-9.]+)s speed_Bps=([0-9.]+) error=(.*)$"
)
for line in Path(sys.argv[1]).read_text(encoding="utf-8").splitlines():
    match = pattern.search(line)
    if not match:
        continue
    path, target, run, ttfb, total, speed, error = match.groups()
    if run == "0":
        continue
    group = groups[(path, target)]
    group["ttfb"].append(float(ttfb))
    group["total"].append(float(total))
    group["speed"].append(float(speed))
    group["errors"] += bool(error.strip())

for key in sorted(groups):
    values = groups[key]
    print(
        "summary path=%s target=%s samples=%d avg_speed_KiBps=%.1f "
        "median_ttfb=%.3fs median_total=%.3fs errors=%d"
        % (
            key[0], key[1], len(values["total"]),
            statistics.mean(values["speed"]) / 1024,
            statistics.median(values["ttfb"]),
            statistics.median(values["total"]),
            values["errors"],
        )
    )
PY
```

Expected: 每个 path/target 组合都有统计行；Cloudflare 与 Google 各有五个正式样本，ChatGPT 各有五个样本。样本不足时不进入 Task 5。

---

### Task 5: 判定瓶颈并形成供应商证据

**Files:**
- Read: `/root/residential-benchmark-20260903.txt`
- Create: `docs/2026-09-03-residential-provider-path-evidence.md`

**Interfaces:**
- Consumes: Task 4 的脱敏出口和统计摘要。
- Produces: 一个明确的诊断分支与可提交给供应商的脱敏证据文档。

- [x] **Step 1: 使用固定判定矩阵选择唯一分支**

Apply exactly one branch:

```text
A. direct Cloudflare 快，residential Cloudflare 慢，residential Google 快
   => 住宅供应商到 Cloudflare/OpenAI 的特定路由或整形问题。

B. direct Cloudflare 与 residential Cloudflare 都慢，但 Google 都快
   => 当前 VPS 到 Cloudflare 的出口路由问题，先处理 VPS 线路。

C. residential Cloudflare 与 residential Google 都慢，direct 两者都快
   => 住宅接入网关、套餐带宽或并发限制问题。

D. direct 与 residential 的 Cloudflare、Google 都慢
   => VPS 总体网络质量问题，不向住宅供应商归因。

E. direct 与 residential 的 Cloudflare、Google 都快，但 Mac 端仍慢
   => 瓶颈位于 Mac 到 VPS 的外层链路，结束供应商路径调查并转入
      Hysteria2 或更优 VPS 跨境线路方案。
```

Expected: 结论必须由同批 VPS 数据支持，不能使用 Mac 端数据替代。

- [x] **Step 2: 写入脱敏证据文档**

Create `docs/2026-09-03-residential-provider-path-evidence.md` with:

```markdown
# 固定住宅代理路径证据

## 约束

- 固定住宅出口必须保持不变。
- 测试未修改服务端或客户端配置。

## 测试方法

- 测试位置：当前 VPS。
- 路径：VPS 直连、VPS 通过现有住宅 SOCKS5。
- 目标：Cloudflare 100 KB、Google CDN 100 KB、ChatGPT 首页。
- 每个组合：五次正式样本。

## 脱敏结果

从 `/root/residential-benchmark-20260903.txt` 复制六条 `summary` 记录和三条出口确认记录；不得复制完整 IP、用户名、密码、UUID 或密钥。

## 判定

复制 Task 5 Step 1 中唯一匹配的判定文字。

## 供应商请求

- 确认保持当前固定出口 IP 时可用的其他接入域名、网关、机房或端口。
- 确认 TCP 并发、单连接带宽、总带宽和流量整形限制。
- 确认到 Cloudflare/OpenAI 是否存在已知路由问题。
- 任何替代接入方式必须先证明出口 IP 不变。
```

Expected: 文档不含完整住宅 IP 或任何凭据，且结论与判定矩阵一致。

- [x] **Step 3: 提交证据文档**

```bash
git add docs/2026-09-03-residential-provider-path-evidence.md
git commit -m "docs: record residential provider path evidence"
```

Expected: commit 只包含该证据文件。

---

### Task 6: 结束检查点

**Files:**
- Read: `/etc/mihomo-residential/config.yaml`
- Read: `/root/residential-benchmark-20260903.txt`

**Interfaces:**
- Consumes: 全部基线与证据文档。
- Produces: 进入“供应商同 IP 网关调整”或“VPS 路由调整”的明确下一步，不直接实施未验证变更。

- [x] **Step 1: 确认测试没有改变运行状态**

```bash
printf 'service='; systemctl is-active mihomo-residential.service
printf 'tcp443='; ss -lntp | awk '$4 ~ /:443$/ {print $4 " " $NF}'
sudo /usr/local/bin/mihomo-residential \
  -t -d /var/lib/mihomo-residential \
  -f /etc/mihomo-residential/config.yaml 2>&1 | tail -1
```

Expected: 服务仍为 `active`、TCP 443 仍监听、配置检查成功。

- [x] **Step 2: 从 Mac 复核固定住宅出口与 ChatGPT 可达性**

```bash
curl --max-time 10 -x http://127.0.0.1:7897 -sS https://api.ipify.org \
  | awk -F. '{print $1 "." $2 ".x.x"}'
curl --max-time 10 -x http://127.0.0.1:7897 -sS -o /dev/null \
  -w 'chatgpt code=%{http_code} tls=%{time_appconnect}s ttfb=%{time_starttransfer}s total=%{time_total}s error=%{errormsg}\n' \
  https://chatgpt.com/
```

Expected: 出口仍为 `204.1.x.x`，ChatGPT 不出现 `code=000`。

- [x] **Step 3: 停在供应商变更前的审批点**

Do not change the upstream hostname, port, credentials, protocol, plan, or fixed IP in this plan. Present the Task 5 conclusion and obtain the user's explicit approval for the exact next change.

Expected: 方案 A 第一阶段以证据和单一诊断分支结束，没有未授权的供应商或生产配置变更。
