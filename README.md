# MeepaGateway

Multi-platform AI bot gateway. Connect AI agents to MeepaChat, Discord, Telegram, Slack, and WhatsApp.

## Install

```bash
curl -fsSL https://meepagateway.bogpad.io/install.sh | sh
```

Or via Homebrew:

```bash
brew install bogpad/tap/meepagateway
```

## Quick start

```bash
meepagateway
```

On first run, you'll choose a setup method:

- **`[1]` Terminal wizard** — interactive step-by-step in your terminal
- **`[2]` Web dashboard** — Captain Dashboard at `http://localhost:8092`

Both walk you through configuring your LLM provider, creating an agent, and connecting to a chat platform. To re-run later: `meepagateway setup`

## Deploy to a server

### DigitalOcean (recommended)

One command — auto-detects SSH keys, prints your dashboard URL and setup code when done:

```bash
bash <(curl -fsSL https://meepagateway.bogpad.io/deploy-do.sh)
```

Skip the setup wizard entirely by passing your API key and password:

```bash
bash <(curl -fsSL https://meepagateway.bogpad.io/deploy-do.sh) \
  --anthropic-key "sk-ant-..." \
  --password "your-password"
```

The script automatically uses a **pre-baked snapshot** if one exists in your account (~30 second boot), otherwise falls back to **cloud-init** on a fresh Ubuntu image (~3 minute install).

> Need doctl? `brew install doctl && doctl auth init`

### Hetzner

```bash
hcloud ssh-key list
# If empty: hcloud ssh-key create --name my-key --public-key-from-file ~/.ssh/id_ed25519.pub

IP=$(hcloud server create --name meepagateway \
  --image "$(hcloud image list -t snapshot -o columns=id,description | grep meepagateway- | sort -t- -k2 -V | tail -1 | awk '{print $1}')" \
  --type cx23 --ssh-key my-key \
  --user-data "$(curl -fsSL https://meepagateway.bogpad.io/cloud-init-image.sh)" \
  -o columns=public_net --no-header | awk '{print $1}')

echo "Dashboard: http://$IP:8092"
```

> Need hcloud? `brew install hcloud` or [install docs](https://github.com/hetznercloud/cli)

### Any VPS (Linode, Vultr, AWS, GCP, Azure, etc.)

Use cloud-init — paste as "User data" when creating a server:

```bash
curl -sfL https://meepagateway.bogpad.io/cloud-init.sh
```

After boot (~3 minutes), open `http://<server-ip>:8092`.

Environment variables you can set before running cloud-init:

| Variable | Description |
|-|-|
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude models |
| `OPENAI_API_KEY` | OpenAI API key (optional) |
| `MEEPAGATEWAY_PASSWORD` | Dashboard password (skip setup wizard) |
| `TAILSCALE_AUTH_KEY` | Tailscale auth key for private HTTPS (optional) |

### Manual install

On any Ubuntu server:

```bash
curl -fsSL https://meepagateway.bogpad.io/install.sh | sh
meepagateway
```

## Docker

```bash
docker run -d \
  --name meepa-gateway \
  -p 8092:8092 \
  -v $(pwd)/meepa.toml:/app/meepa.toml:ro \
  -e ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}" \
  ghcr.io/bogpad/meepagateway:latest
```

## Updating

Built-in self-update — no need to re-download or redeploy:

```bash
meepagateway update
```

Or update via the Captain Dashboard (Settings > Update).

Other methods: `brew upgrade meepagateway` | `docker pull ghcr.io/bogpad/meepagateway:latest` | re-run the install script

## Documentation

Full docs: [meepagateway/docs](https://github.com/bogpad/meepa/tree/main/meepagateway/docs)
