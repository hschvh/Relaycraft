# Hysteria2 A/B Testing

Use this file only after the baseline VLESS path is already stable and you have a reason to compare Hysteria2 fairly.

## Preconditions

- A valid certificate path is available.
- UDP 443 is actually permitted end to end.
- The VLESS fallback remains intact.
- The comparison uses the same exit, same target, and same payload.

If any precondition is missing, stop and keep VLESS as the only supported path until the missing capability is verified. Do not claim a Hysteria2 result.

## Measurement table

Record multiple samples for each branch:

| Branch | RTT | TLS / handshake | TTFB | Total | Throughput | Worst sample |
| --- | --- | --- | --- | --- | --- | --- |
| VLESS | <value> | <value> | <value> | <value> | <value> | <value> |
| Hysteria2 | <value> | <value> | <value> | <value> | <value> | <value> |

Take five or more repeated samples. A single low-latency result is not enough.

## Network experiments

Temporary qdisc, BBR, and related tuning experiments must include immediate restoration commands. If the new branch fails, switch back to the preserved VLESS path first.

## Configuration truth rule

Do not infer BBR or Brutal conclusions from marketing names or client flags. Base the conclusion on the actual server configuration and the observed measurements.

## Recovery rule

If the comparison is not apples-to-apples, if UDP is blocked, or if the fallback path was changed during the test, stop, restore the fallback, and rerun with a clean setup.
