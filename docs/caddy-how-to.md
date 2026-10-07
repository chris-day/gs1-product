# Caddy How-To

Use Caddy to expose the local Zensical preview at `https://gs1-product.perdl.com`.
Caddy accepts HTTPS on port 443 and forwards requests to Zensical over HTTP on
`localhost:8000`. Run all project commands below from the repository root.

## Install Caddy

On Debian or Ubuntu, install Caddy from its official stable repository:

```bash
sudo apt update
sudo apt install --yes debian-keyring debian-archive-keyring apt-transport-https curl gnupg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo chmod o+r /usr/share/keyrings/caddy-stable-archive-keyring.gpg
sudo chmod o+r /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
caddy version
```

See the [official installation instructions](https://caddyserver.com/docs/install#debian-ubuntu-raspbian)
for details and other operating systems. Caddy is a system executable; it is not
installed into the Python virtual environment.

The Debian/Ubuntu package starts a `caddy` systemd service automatically. For the
manual foreground workflow in this guide, stop and disable that service so it
does not compete for the same ports (only do this if it is not serving other sites):

```bash
sudo systemctl disable --now caddy
```

## Configure the reverse proxy

Create `caddy-reverse-proxy.json` in the repository root using this example:

```json
{
  "apps": {
    "http": {
      "servers": {
        "zensical_server": {
          "listen": [
            ":443"
          ],
          "routes": [
            {
              "match": [
                {
                  "host": [
                    "gs1-product.perdl.com"
                  ]
                }
              ],
              "handle": [
                {
                  "handler": "reverse_proxy",
                  "upstreams": [
                    {
                      "dial": "localhost:8000"
                    }
                  ]
                }
              ]
            }
          ]
        }
      }
    },
    "tls": {
      "certificates": {
        "load_files": [
          {
            "certificate": "<CERTIFICATE_BUNDLE_PATH>",
            "key": "<PRIVATE_KEY_PATH>",
            "tags": [
              "custom_certs"
            ]
          }
        ]
      }
    }
  }
}
```

Replace these literal placeholders before running Caddy:

| Placeholder | Value |
| --- | --- |
| `<CERTIFICATE_BUNDLE_PATH>` | Absolute path to your PEM certificate bundle, including the intermediate certificates. |
| `<PRIVATE_KEY_PATH>` | Absolute path to the matching PEM private key, without passphrase encryption. |

These are values to edit in the JSON, not shell environment variables. Caddy must
be able to read both files. Keep the private key accessible only to authorized
users. See Caddy's [certificate file reference](https://caddyserver.com/docs/json/apps/tls/certificates/load_files/).

The certificate must cover `gs1-product.perdl.com`, and clients must trust its issuer.
Ensure that the hostname resolves to the machine running Caddy and that clients
can reach port 443. For another hostname, update the `host` array and use a
certificate covering that hostname. Caddy may also open port 80 for automatic
HTTP-to-HTTPS redirects; leave it available if using those redirects.

The local `caddy-reverse-proxy.json` is ignored by Git. Keep your real certificate
and key paths in that local file. Renew manually supplied certificates through
your certificate provider and restart Caddy after replacing them.

## Start Zensical

In one terminal, start the local preview using the project virtual environment:

```bash
.venv/bin/zensical serve --dev-addr localhost:8000
```

Leave this terminal running. Zensical builds the documentation and serves it on
the loopback interface. A standalone `zensical build` creates static files and
does not start the HTTP server required by this reverse proxy.

Check the upstream in another terminal:

```bash
curl -I http://localhost:8000/
```

## Validate and run Caddy

After replacing the placeholders, validate the configuration:

```bash
sudo caddy validate --config caddy-reverse-proxy.json
```

Then run Caddy in a second terminal:

```bash
sudo caddy run --config caddy-reverse-proxy.json
```

This runs Caddy in the foreground. Leave both terminals open, then visit
[https://gs1-product.perdl.com](https://gs1-product.perdl.com), or check with:

```bash
curl -I https://gs1-product.perdl.com/
```

Press `Ctrl+C` in each terminal to stop its server. After editing the JSON,
validate it again and restart Caddy. See the
[Caddy command-line reference](https://caddyserver.com/docs/command-line) for
validation and foreground execution details.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| `502 Bad Gateway` | Confirm Zensical is running and `curl -I http://localhost:8000/` succeeds. |
| Address already in use | Check for the packaged Caddy service or another server using ports 80, 443, or Caddy's default admin port 2019. |
| Certificate loading error | Replace both placeholders; check file paths, readability, PEM format, and that the certificate and key match. |
| Browser certificate warning | Check the certificate hostname, expiry, intermediate chain, and client trust. |
| Connection timeout | Check hostname resolution, firewall rules, and reachability of port 443. |
