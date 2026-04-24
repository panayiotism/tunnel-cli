# tunnel-cli

A free, self-contained ngrok replacement built on [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/). Two small shell scripts that give you an ngrok-style workflow with **no account fees, no bandwidth caps, and optional custom domains** on your own Cloudflare-hosted domain.

## Why

ngrok's free tier is rate-limited and the random URL changes every restart. The paid plans start around $8/mo just for a stable URL. Cloudflare Tunnel gives you the same thing — plus unlimited bandwidth, HTTPS, and your own domain — for free.

| | ngrok free | ngrok paid | tunnel-cli (this) |
|---|---|---|---|
| Cost | free | $8–20+/mo | **free** |
| Bandwidth | limited | limited | **unlimited** |
| Stable URL | no | yes | **yes** (named mode) |
| Custom domain | no | yes | **yes** (named mode) |
| HTTPS | yes | yes | **yes** |

## What's in the box

- **`tunnel <port>`** — quick ephemeral tunnel with a random `*.trycloudflare.com` URL. Zero config, no account needed. The ngrok-style one-liner.
- **`tunnel-serve <subdomain.yourdomain.com> <port>`** — named tunnel on your own Cloudflare-hosted domain. Stable URL, reusable across runs. Ideal for client demos and webhooks.

## Install

### 1. Install cloudflared

Pick whichever works for your system. Both scripts just need `cloudflared` on `$PATH`.

**Linux / WSL2 (no sudo, portable):**
```bash
mkdir -p ~/.local/bin
curl -fsSL -o ~/.local/bin/cloudflared \
  https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
chmod +x ~/.local/bin/cloudflared
```

Make sure `~/.local/bin` is on your `$PATH` (it usually is on Ubuntu). Verify with `cloudflared --version`.

**Debian / Ubuntu (apt repo):**
```bash
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt-get update && sudo apt-get install -y cloudflared
```

**macOS:**
```bash
brew install cloudflared
```

**Other platforms:** see the [official install docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/).

### 2. Install the scripts

Clone this repo and symlink the two scripts into a directory on your `$PATH`:

```bash
git clone https://github.com/panayiotism/tunnel-cli.git
cd tunnel-cli
ln -sfn "$PWD/tunnel"       ~/.local/bin/tunnel
ln -sfn "$PWD/tunnel-serve" ~/.local/bin/tunnel-serve
```

Now `tunnel` and `tunnel-serve` work from anywhere.

## Usage

### Quick ephemeral tunnel

One-liner, no account, random URL — best for quick shares and webhook testing:

```bash
tunnel 3000
```

Output:
```
→ Starting quick tunnel to http://localhost:3000
  (Ctrl-C to stop)

▶ Public URL: https://random-words-here.trycloudflare.com
```

Hit Ctrl-C to stop. The URL disappears when the process exits.

**Optional: bind to a specific host**
```bash
tunnel 3000 127.0.0.1
```

### Named tunnel (stable URL on your own domain)

Prerequisite: a domain on Cloudflare (free plan is fine).

**First-time setup** — one-time browser login to associate cloudflared with your Cloudflare account:

```bash
tunnel-serve demo.yourdomain.com 3000
```

It detects no cert exists, runs `cloudflared tunnel login`, and prints an auth URL. Open it in a browser, pick your domain, authorize. A cert is written to `~/.cloudflared/cert.pem`. The script exits.

**Every run after that** — same command:

```bash
tunnel-serve demo.yourdomain.com 3000
```

It creates the tunnel (if missing), upserts the DNS CNAME on Cloudflare, and serves `http://localhost:3000` at `https://demo.yourdomain.com`. Ctrl-C to stop. Re-run anytime to bring the tunnel back up at the same URL.

**Use multiple subdomains** for different projects or clients:
```bash
tunnel-serve staging.yourdomain.com 3000    # project A
tunnel-serve demo.yourdomain.com 5173       # project B
tunnel-serve client-acme.yourdomain.com 8080 # specific client
```

Each gets its own tunnel and DNS record, reused across runs.

## Common gotchas

**DNS filters (AdGuard, NextDNS, Pi-hole, some corporate resolvers) block `*.trycloudflare.com`.** If a quick tunnel URL returns `ERR_NAME_NOT_RESOLVED`, the issue is almost always DNS filtering, not the tunnel itself. Check with `nslookup <the-url> 1.1.1.1` — if that works but your system resolver doesn't, your DNS filter is dropping it. Fix: either allowlist `trycloudflare.com`, or use a named tunnel on your own domain (which won't be filtered).

**First-run browser login for named tunnels needs a browser.** On a headless machine or WSL2, copy the printed auth URL into a browser on any machine logged into the same Cloudflare account. You only do this once per machine.

**`--overwrite-dns` is used for CNAME creation.** If `demo.yourdomain.com` already has an A/AAAA record, the script will overwrite it. Use a subdomain that isn't in use for production.

**Credentials live outside the repo.** `~/.cloudflared/cert.pem` and tunnel JSON credentials are written to your home directory — never to this repo. `.gitignore` guards against accidental commits anyway.

## For AI coding agents

A `SKILL.md` file at the root of this repo describes when and how to reach for these tools. Agents like Claude Code can load it directly so they know to use `tunnel` / `tunnel-serve` instead of suggesting paid tunneling services.

## License

MIT — see [LICENSE](./LICENSE).
