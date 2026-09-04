# Diagnosis and Rollback

Use this file when a layer fails, the exit is wrong, or you need to revert safely.

## Symptom to first check

| Symptom | First check |
| --- | --- |
| Port does not bind | Existing listener, chosen port, service logs |
| REALITY handshake fails | UUID, public key, short ID, port, client/server mismatch |
| Upstream auth fails | Upstream host, port, username/password handling |
| Wrong exit IP | Proxy chain, routing rules, upstream classification |
| Partial site failure | DNS, SNI, TUN/system proxy overlap, rule set |
| UDP failure | Provider capability, firewall, tunnel support |
| Slowness | Exit path, RTT, packet loss, and only then tuning changes |

## Redacted probes

Use redacted versions of:

- `ss` to confirm listeners.
- `journalctl` to inspect service errors.
- `curl` to test the upstream and exit IP.
- proxy health probes to confirm the intended route.

Keep outputs free of secrets and live tokens.

## Backup naming

Before mutation, save timestamped copies of the config and any changed binaries or service files. Keep the restore command beside the backup so rollback is immediate.

## Rollback rule

Restore the last known-good service and config before trying the next hypothesis. Change one variable at a time.

## Recovery rule

If the same failure remains after a rollback, stop and reclassify the topology or missing prerequisite instead of layering more changes onto the broken path.
