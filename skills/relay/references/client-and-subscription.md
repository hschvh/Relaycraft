# Client and Subscription

Use this reference for an authorized Mihomo/Clash client after the server-side baseline has passed its gates.

## Offline placeholder configuration

Validate a local copy before import. Keep UUIDs, public keys, Short IDs, subscription tokens, and URLs out of chat and Git; the placeholders below are not deployable values.

```yaml
proxies:
  - name: <NODE_NAME>
    type: vless
    server: <VPS_PUBLIC_IP>
    port: <TCP_PORT>
    uuid: <NEW_UUID>
    tls: true
    servername: <REALITY_TARGET_HOST>
    reality-opts:
      public-key: <SERVER_PUBLIC_KEY>
      short-id: <NEW_SHORT_ID>
    udp: false

proxy-groups:
  - name: <PROXY_GROUP>
    type: select
    proxies: [<NODE_NAME>, DIRECT]

rules:
  - DOMAIN-SUFFIX,<TARGET_DOMAIN>,<PROXY_GROUP>
  - MATCH,DIRECT
```

Use the installed client version's documented parser in offline mode, for example its `-t`/`--test` option. Do not import an unverified subscription into a production profile. If parsing fails, stop and correct syntax before network tests.

## TUN and system proxy

TUN captures traffic at the interface layer; a system proxy changes applications that honor proxy settings. Enable only the layer required by the current test. Running both can create loops, duplicate DNS handling, or make an apparent latency problem look like a server failure. Record which mode was active for every sample.

## Measured toggles

Treat IPv6 and `tcp-concurrent` as experiments, not universal fixes. Capture the baseline, change one toggle, repeat the same target and payload, and revert if it does not improve the agreed metrics. Keep `udp: false` unless the full upstream path is known to support UDP.

## File permissions and subscriptions

Store local configs with restrictive permissions (for example `chmod 600 <CONFIG_FILE>`), ensure the client process can read them, and never place a live subscription URL or token in shell history, issue text, logs, or screenshots. When a subscription is required, import it through the client's normal authenticated UI or a protected file, then remove temporary copies according to the client's documented procedure.

## Import and exit verification

1. Back up the last known-good client profile.
2. Run offline syntax validation and inspect the selected proxy/group/rules.
3. Import or reload the profile through the client UI/CLI and confirm the expected node is selected.
4. Verify an HTTPS request and the redacted exit IP through the proxy; verify DNS behavior separately.
5. Confirm the existing direct route still works, then test restart recovery.

If the exit is the VPS datacenter address instead of the intended residential address, stop at routing/upstream diagnosis. Do not compensate by enabling TUN, IPv6, concurrent connections, or UDP blindly. Restore the previous profile if the import or health check fails.
