# Residential Upstream

Use this file when the residential resource is an upstream proxy that sits between the VPS and the internet.

## Supported placeholder shapes

```yaml
proxies:
  - name: <RES_HTTP>
    type: http
    server: <RES_HOST>
    port: <RES_PORT>
    username: <RES_USER>
    password: <RES_PASSWORD>
```

```yaml
proxies:
  - name: <RES_SOCKS5>
    type: socks5
    server: <RES_HOST>
    port: <RES_PORT>
    username: <RES_USER>
    password: <RES_PASSWORD>
```

```yaml
proxies:
  - name: <RES_HTTPS>
    type: http
    server: <RES_HOST>
    port: <RES_PORT>
    tls: true
```

Use the provider's documented authentication variant and do not assume one type implies another.

## Exit-IP check

Validate the upstream by making an interactive `curl` request through the proxy and confirming the returned exit IP matches the purchased residential IP. Repeat the test more than once before trusting the result.

## UDP rule

Keep `udp: false` by default until the provider documents UDP support or you have measured it. Setting `udp: true` on the client does not magically turn ordinary TCP web requests into UDP.

## Static-exit rule

If the service is sold as a static residential IP, confirm the exit remains stable across several checks. If the exit drifts or changes unexpectedly, stop and classify the product again before continuing.

## Recovery rule

If authentication fails, the exit IP is wrong, or the provider's capabilities are unclear, stop before layering the proxy into a node configuration.
