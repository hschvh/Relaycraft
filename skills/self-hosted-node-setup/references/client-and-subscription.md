# Client and Subscription

Use this file for Mihomo client setup, subscription import, and client-side verification.

## Client placeholders

Keep proxy, proxy-group, and rule examples generic:

```yaml
proxies:
  - name: <NODE>
    type: vless
    server: <VPS_PUBLIC_IP>
```

```yaml
proxy-groups:
  - name: <PROXY_GROUP>
    type: select
    proxies:
      - <NODE>
      - DIRECT
```

```yaml
rules:
  - MATCH,<PROXY_GROUP>
```

## TUN vs system proxy

Treat TUN mode and system proxy as different routing layers. Enable only the one you need for the current check unless the test explicitly requires both.

## Measured toggles

IPv6 and `tcp-concurrent` are measured toggles, not universal fixes. Change one variable at a time and keep the baseline available for comparison.

## Subscription hygiene

- Validate the config offline before import.
- Check file permissions before exposing a subscription file.
- Import only the non-secret data needed for the client.
- Do not paste live subscription URLs or tokens into chat.

## Exit-IP verification

After import, confirm the chosen proxy really exits from the intended address. If the exit is wrong, stop and diagnose the upstream or routing layer before changing more client options.

## Recovery rule

If the client parses but traffic still fails, return to the last known-good baseline and isolate the smallest changed option.
