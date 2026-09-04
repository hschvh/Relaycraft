# VLESS + REALITY with Mihomo

Use this reference for the baseline path after authorization, topology classification, and read-only preflight are complete. All commands are templates: replace placeholders only on the authorized host and keep their values out of chat and Git.

## Read-only preflight

Run and save redacted output before mutation:

```sh
uname -m
timedatectl show -p NTPSynchronized --value
df -h /
sudo ss -ltnup
sudo systemctl --type=service --state=running --no-pager
sudo ufw status verbose                 # or inspect the provider security group
```

Confirm the candidate TCP port is unused and that Nginx, Caddy, Xray, x-ui, and other existing services remain healthy. If 443 is occupied, choose an unused TCP port such as `<TCP_PORT>` and open only that port in the authorized firewall. Never stop an occupant to make the example fit.

## Architecture-aware binary verification

Determine the host architecture first, then download the provider's documented Mihomo build for that architecture. Verify the release checksum/signature and inspect it without launching:

```sh
file /tmp/<MIHOMO_BINARY>
sha256sum /tmp/<MIHOMO_BINARY>       # compare with the documented release checksum
install -m 0755 /tmp/<MIHOMO_BINARY> /usr/local/bin/<MIHOMO_BINARY_NAME>
/usr/local/bin/<MIHOMO_BINARY_NAME> version
```

If the architecture, checksum, or version check does not match, stop before installing or creating a service.

## Fresh credential generation

Generate a new UUID, a new REALITY X25519 keypair, and a new random Short ID for every deployment using the Mihomo release's documented generator. Keep the private key in a root-readable server file only; deliver the public key and non-secret parameters to the client. Never reuse values from examples or another node.

Use the installed release's documented generator or equivalent local tools, with output redirected to a restricted file rather than chat. A generic placeholder pattern is:

```sh
umask 077
<MIHOMO_BINARY_NAME> generate uuid > /tmp/<UUID_FILE>
<MIHOMO_BINARY_NAME> generate reality-keypair > /tmp/<REALITY_KEYPAIR_FILE>
head -c <SHORT_ID_BYTES> /dev/urandom | od -An -tx1 | tr -d ' \\n' > /tmp/<SHORT_ID_FILE>
test -s /tmp/<UUID_FILE> && test -s /tmp/<REALITY_KEYPAIR_FILE> && test -s /tmp/<SHORT_ID_FILE>
```

Subcommands and output formats vary by release: check `<MIHOMO_BINARY_NAME> help` and the signed release documentation before running. Verify that the UUID and Short ID have the expected format, that the keypair contains distinct public/private values, and then move only the private value to a root-readable server file. Never print these files or commit them.

## Placeholder server configuration

Create a restricted server config only after backing up any existing config. The private key placeholder below is server-only:

```yaml
log:
  level: info
inbounds:
  - type: vless
    tag: vless-in
    listen: 0.0.0.0
    port: <TCP_PORT>
    users:
      - name: <NODE_USER>
        uuid: <NEW_UUID>
    tls:
      enabled: true
      reality:
        enabled: true
        handshake:
          server: <REALITY_TARGET_HOST>
          server_port: 443
        private_key: <SERVER_PRIVATE_KEY>
        short_id: <NEW_SHORT_ID>
outbounds:
  - type: direct
    tag: direct
```

Use the exact field spelling supported by the installed Mihomo version; validate before restart. If a residential upstream is part of the topology, configure it as a separate outbound and route only after the direct baseline parses and listens.

## Placeholder client configuration

The client receives only the public key:

```yaml
proxies:
  - name: <NODE_NAME>
    type: vless
    server: <VPS_PUBLIC_IP>
    port: <TCP_PORT>
    uuid: <NEW_UUID>
    tls: true
    servername: <REALITY_TARGET_HOST>
    client-fingerprint: chrome
    reality-opts:
      public-key: <SERVER_PUBLIC_KEY>
      short-id: <NEW_SHORT_ID>
    udp: false
```

Do not copy `<SERVER_PRIVATE_KEY>` into the client. Keep `udp: false` until the upstream and full path have been independently proven to support UDP.

## systemd and verification gates

Back up the unit and config before mutation, then create a dedicated service with restricted permissions:

```ini
[Unit]
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/<MIHOMO_BINARY_NAME> -d /etc/<MIHOMO_DIR> -f /etc/<MIHOMO_DIR>/config.yaml
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

Run the installed binary's documented config-check command, then:

```sh
sudo systemctl daemon-reload
sudo systemctl enable --now <MIHOMO_UNIT>
sudo systemctl is-active --quiet <MIHOMO_UNIT> && echo active
sudo ss -ltnp | rg ':<TCP_PORT>\\b'
sudo journalctl -u <MIHOMO_UNIT> -n 80 --no-pager | sed -E 's/(uuid|private[_-]?key|short[_-]?id|password|token):[^ ]+/***REDACTED***/Ig'
```

Proceed in layers: config parse, systemd active, listener present, VLESS/REALITY handshake, HTTPS status, exit IP/DNS, and restart recovery. If any gate fails, stop at that layer, collect redacted evidence, restore the last known-good backup if needed, and do not add an upstream or performance experiment.
