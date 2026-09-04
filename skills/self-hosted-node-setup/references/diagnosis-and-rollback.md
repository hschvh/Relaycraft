# Diagnosis and Rollback

Use this reference whenever a verification layer fails, an exit is wrong, or a performance experiment needs to be undone. Diagnose from redacted evidence and change one variable at a time.

## Symptom-to-first-check matrix

| Symptom | First check | Stop/rollback gate |
| --- | --- | --- |
| Port does not bind | `ss` listener, port ownership, firewall, and systemd status | Do not stop the occupant; choose an unused port or restore the unit |
| REALITY handshake fails | Server/client UUID, public key, Short ID, SNI/target, and port correspondence | Stop before adding upstreams; rotate credentials if exposed |
| Upstream authentication fails | Provider auth variant, allowlist, TLS/SNI, clock, and redacted request result | Restore direct baseline; do not paste the password into diagnostics |
| Wrong exit IP | VPS outbound route, proxy chain, rule order, and independent upstream exit test | Stop before enabling TUN/IPv6/UDP; restore the known-good route |
| Partial site failure | DNS mode, SNI, MTU/MSS evidence, TUN/system-proxy overlap, and rule match | Revert the last client or network toggle, then retest one target |
| UDP failure | Provider documentation, firewall, listener, and end-to-end UDP probe | Keep UDP disabled; use VLESS/TCP fallback |
| Slowness or packet loss | Same-exit RTT, loss, handshake, TTFB, total time, throughput, and worst sample | Do not claim a faster protocol from one latency number; revert the last experiment |

## Redacted evidence probes

Run only on the authorized host and redact before sharing:

```sh
sudo ss -ltnup | sed -E 's/([0-9a-fA-F:.]{3,})/<ADDRESS>/g'
sudo systemctl status <MIHOMO_UNIT> --no-pager
sudo journalctl -u <MIHOMO_UNIT> -n 100 --no-pager \
  | sed -E 's/(uuid|private[_-]?key|short[_-]?id|password|token|subscription[^ ]*):?[^ ]+/***REDACTED***/Ig'
curl --silent --show-error --fail --proxy "http://<RES_HOST>:<RES_PORT>" https://<IP_CHECK_HOST>/ \
  | sed -E 's/[0-9a-fA-F:.]+/<REDACTED_IP>/g'
```

Use proxy health checks that identify the route without returning a full credential-bearing URL. Never include raw config, command history, process arguments, subscription links, or private keys in a ticket or report.

## Backup before mutation

Before editing a config, unit, binary, qdisc, BBR, MTU, or client profile, create a restricted timestamped backup and record its restore command:

```sh
STAMP="<UTC_TIMESTAMP>"
if [ -f /etc/<MIHOMO_DIR>/config.yaml ]; then
  sudo install -m 600 /etc/<MIHOMO_DIR>/config.yaml "/var/backups/<MIHOMO_DIR>-config-${STAMP}.yaml"
else
  printf '%s\\n' 'No existing Mihomo config; record this before creating a new one.'
fi
if [ -f /etc/systemd/system/<MIHOMO_UNIT>.service ]; then
  sudo cp /etc/systemd/system/<MIHOMO_UNIT>.service "/var/backups/<MIHOMO_UNIT>-${STAMP}.service"
else
  printf '%s\\n' 'No existing unit; record this before creating a new one.'
fi
# Record the exact, reviewed restore command next to these files.
```

Backups must not contain live values in Git. Store them only in the authorized, access-controlled location.

## Service and config rollback

If a change fails a gate, stop the service cleanly, restore the last known-good config and unit, reload systemd, and re-run the lower-layer checks before attempting anything else:

```sh
sudo systemctl stop <MIHOMO_UNIT>
sudo install -m 600 "/var/backups/<MIHOMO_DIR>-config-<UTC_TIMESTAMP>.yaml" /etc/<MIHOMO_DIR>/config.yaml
sudo install -m 644 "/var/backups/<MIHOMO_UNIT>-<UTC_TIMESTAMP>.service" /etc/systemd/system/<MIHOMO_UNIT>.service
sudo systemctl daemon-reload
sudo systemctl start <MIHOMO_UNIT>
sudo systemctl is-active <MIHOMO_UNIT>
```

For qdisc/BBR/MTU experiments, restore the captured values immediately. For client changes, restore the previous profile and disable the new proxy before retesting. Preserve the VLESS fallback whenever Hysteria2 is being evaluated.

## One-variable rule

Record the hypothesis, exact change, timestamp, test target, exit identity, and result. Change one variable per trial. If the same failure remains after rollback, stop and reclassify the topology or missing prerequisite instead of layering more guesses onto a broken path.
