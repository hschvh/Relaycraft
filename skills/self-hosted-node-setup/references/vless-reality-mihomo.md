# VLESS + REALITY with Mihomo

Use this file for the baseline deployment path after intake and preflight are complete.

## Read-only preflight

Before changing anything, inspect:

- Current time sync state.
- Existing listeners on candidate ports.
- Firewall or cloud security-group rules.
- Disk space and service health.
- Any already-running Nginx, Caddy, Xray, x-ui, or similar services.

If 443 is occupied, choose a free port. Do not stop the existing service unless the user explicitly asks.

## Binary and architecture checks

Verify the Mihomo binary matches the host architecture before launch. Keep the check read-only until the correct package is confirmed.

## Fresh credentials

Generate a new UUID, a new REALITY keypair, and a new Short ID for each deployment.

- The server keeps the private key.
- The client receives only the public key and other non-secret parameters.

Never reuse examples from another deployment.

## Placeholder configuration pattern

```yaml
inbounds:
  - type: vless
    listen: 0.0.0.0
    port: <TCP_PORT>
    users:
      - uuid: <UUID>
    tls:
      enabled: true
      reality:
        enabled: true
        private-key: <SERVER_PRIVATE_KEY>
        short-id: <SHORT_ID>

outbounds:
  - type: direct
```

```yaml
proxies:
  - name: <NODE_NAME>
    type: vless
    server: <VPS_PUBLIC_IP>
    port: <TCP_PORT>
    uuid: <UUID>
    tls: true
    reality-opts:
      public-key: <SERVER_PUBLIC_KEY>
      short-id: <SHORT_ID>
```

## systemd and checks

Install the service only after the config parses cleanly. Then verify:

- the unit is active,
- the expected port is listening,
- the log does not contain config syntax errors,
- the client can reach the baseline endpoint.

## Recovery rule

If the config fails to parse, the listener does not appear, or the handshake fails, stop and fix that layer before adding residential upstreams or performance experiments.
