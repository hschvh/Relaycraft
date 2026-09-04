# Residential Upstream

Use this reference when the residential product is an upstream HTTP, HTTPS, or SOCKS5 proxy. Validate this hop independently before routing a VLESS node through it.

## Mihomo placeholder variants

Keep credentials in an interactive prompt or restricted runtime file. The following structures are examples only:

```yaml
proxies:
  - name: <RES_HTTP>
    type: http
    server: <RES_HOST>
    port: <RES_PORT>
    username: <RES_USER>
    password: <UPSTREAM_PASSWORD>
    udp: false
```

```yaml
proxies:
  - name: <RES_HTTPS>
    type: http
    server: <RES_HOST>
    port: <RES_PORT>
    tls: true
    sni: <RES_SNI>
    username: <RES_USER>
    password: <UPSTREAM_PASSWORD>
    udp: false
```

```yaml
proxies:
  - name: <RES_SOCKS5>
    type: socks5
    server: <RES_HOST>
    port: <RES_PORT>
    username: <RES_USER>
    password: <UPSTREAM_PASSWORD>
    udp: false
```

Some providers encode credentials in a URL, require an IP allowlist, or use a separate TLS/SNI value. Follow the provider's documented authentication variant; never guess by moving a password into a command line or URL. If the product is token-based, request only the fact that token authentication exists and enter the token interactively on the authorized host.

## Interactive exit-IP test

Run on the authorized VPS with a prompt or protected environment variable so the secret is not shell history or process arguments:

```sh
read -r -s UPSTREAM_PASSWORD
export UPSTREAM_PASSWORD
curl --silent --show-error --fail \
  --proxy "http://<RES_USER>:${UPSTREAM_PASSWORD}@<RES_HOST>:<RES_PORT>" \
  https://<IP_CHECK_HOST>/ | sed -E 's/[0-9a-fA-F:.]+/<REDACTED_IP>/g'
unset UPSTREAM_PASSWORD
```

Confirm the returned address is the purchased residential exit, repeat at least three times, and compare timestamps. Do not paste the raw response or proxy URL into chat. If authentication fails, first verify the provider's auth variant, allowlist, TLS requirement, and clock; then stop before changing the node.

## Static-exit and capability checks

- A product described as static must return the same redacted exit identity across repeated checks and after a reconnect. If it drifts, reclassify the product and stop.
- Set `udp: false` by default. Enable UDP only after provider documentation and an end-to-end probe both confirm support.
- Setting `udp: true` on a client proxy does not turn ordinary TCP web requests into UDP and does not prove that the upstream carries arbitrary UDP.
- Record documented bandwidth, concurrency, expiry, and destination restrictions as unknown until verified; do not infer them from a plan name.

## Routing and recovery

Keep the upstream as a distinct outbound. Route a small, explicit test rule through it first; leave the direct/VLESS baseline available. If the upstream exit is wrong, unstable, or unauthenticated, restore the last known-good config and stop. Do not add Hysteria2, TUN, or qdisc changes while this layer is failing.
