# Hysteria2 A/B Testing

Use this optional reference only after the direct VLESS + REALITY baseline is stable through handshake, HTTPS, exit-IP, and restart gates. It is an experiment, not a guaranteed speed upgrade.

## Preconditions and guardrails

- The authorized server has a valid certificate and a documented, readable certificate/key path.
- UDP 443 is allowed by the VPS firewall, provider security group, and network path; verify it with an end-to-end probe rather than assuming it.
- The existing VLESS listener and its backup remain available as the immediate fallback.
- Both branches use the same VPS, residential exit, target hostname, payload size, client location, and test window.
- Do not expose or paste certificate private keys, auth passwords, UUIDs, or tokens. Use restricted files and redacted logs.

If any precondition is unknown or false, stop and keep VLESS as the supported path. Do not claim Hysteria2 is faster because a client UI shows a smaller ping.

## Comparable test table

Run at least five repetitions per branch, discard no worst sample, and record the test timestamp and path ID:

| Sample | Branch | RTT | TLS/handshake | TTFB | Total | Throughput | Worst-sample notes |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | --- |
| 1 | VLESS | `<ms>` | `<ms>` | `<ms>` | `<ms>` | `<Mbit/s>` | `<note>` |
| 1 | Hysteria2 | `<ms>` | `<ms>` | `<ms>` | `<ms>` | `<Mbit/s>` | `<note>` |
| ... | ... | ... | ... | ... | ... | ... | ... |

Use the same command, URL, response size, concurrency, and timeout for every sample. Report median and worst values for RTT, handshake, TTFB, total time, and throughput. A result is inconclusive when exits, targets, payloads, or server settings differ.

## Truthful server configuration comparison

Record the actual server settings before testing: congestion control, qdisc, MTU/MSS, certificate path, UDP listener, and any Hysteria2 bandwidth/Brutal parameters. A product label or client option is not evidence that BBR or Brutal is active. Conclusions must cite the server configuration and the repeated measurements.

## Temporary qdisc/BBR experiment

Apply one reversible variable at a time, capture the current values first, and restore immediately after the sample window or on failure:

```sh
OLD_QDISC="$(sysctl -n net.core.default_qdisc)"
OLD_CC="$(sysctl -n net.ipv4.tcp_congestion_control)"
sudo sysctl -w net.core.default_qdisc=<TEST_QDISC>
sudo sysctl -w net.ipv4.tcp_congestion_control=<TEST_CC>
# run the same five-or-more samples and record evidence
sudo sysctl -w net.core.default_qdisc="$OLD_QDISC"
sudo sysctl -w net.ipv4.tcp_congestion_control="$OLD_CC"
```

Do not persist experimental values until the evidence is reviewed. If the command fails, the path degrades, or a service is affected, restore the captured values and switch back to VLESS before collecting more data.

## Decision and rollback

Choose Hysteria2 only when the comparison is repeatable, same-exit, same-target, and materially better across the metrics rather than one sample. Preserve the VLESS node in the client proxy group. On any regression, disable the Hysteria2 route, restore the previous service/config backup, and re-run the VLESS verification gates. Record what changed and restore one variable at a time.
