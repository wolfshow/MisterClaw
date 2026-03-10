# MisterClaw: World's first PicoClaw AI assistant on MiSTer FPGA (DE10-Nano)

<p align="center">
  <img src="MisterClaw.png" alt="MisterClaw" width="720">
</p>

Run an ultra-lightweight AI assistant directly on your MiSTer FPGA (DE10-Nano) to help you manage settings, get game recommendations, troubleshoot issues, and chat — all from the device itself.

---

## What is PicoClaw?

[PicoClaw](https://github.com/sipeed/picoclaw) is an open-source, ultra-lightweight AI assistant written in Go. It runs on minimal hardware — under 10MB RAM, boots in under 1 second — making it a perfect fit for the ARM Linux side (HPS) of the MiSTer FPGA DE10-Nano board.

## What is MiSTer FPGA?

[MiSTer FPGA](https://github.com/MiSTer-devel) is an open-source project that uses the DE10-Nano FPGA board to recreate classic computers, consoles, and arcade machines in hardware. The DE10-Nano runs a full Linux system on its ARM HPS processor — which is where PicoClaw lives.

---

## Why Run PicoClaw on MiSTer?

- Ask questions about MiSTer settings and `.ini` configuration files
- Get game and core recommendations
- Troubleshoot video, audio, and controller issues
- Use it as a general AI assistant running on the device itself
- Future potential: integrate with other hardware connected to MiSTer

---

## Requirements

| Item | Details |
|------|---------|
| Hardware | MiSTer FPGA (DE10-Nano board) |
| MiSTer OS | Standard MiSTer Linux (ARMv7 HPS) |
| Storage | ~20MB free space on the SD card |
| Network | MiSTer connected to your local network (Ethernet recommended) |
| SSH access | Enabled on MiSTer (default: enabled) |
| API key | At least one: OpenAI, Anthropic, OpenRouter, or LiteLLM endpoint |
| PC/Mac | To SSH into MiSTer during setup |

---

## Installation

### Step 1 — SSH into your MiSTer

From your PC or Mac, open a terminal and connect:

```bash
ssh root@MiSTer
```

> Default password is `1` unless you changed it.
> If that doesn't resolve, use your MiSTer's IP address instead: `ssh root@192.168.x.x`

---

### Step 2 — Download the PicoClaw ARM binary

PicoClaw provides precompiled binaries for ARM. The DE10-Nano HPS runs 32-bit ARMv7 Linux.

```bash
cd /media/fat/Scripts

# Download the latest ARMv7 release
curl -L https://github.com/sipeed/picoclaw/releases/latest/download/picoclaw_Linux_armv7.tar.gz -o picoclaw_Linux_armv7.tar.gz

# Extract the binary
tar xzf picoclaw_Linux_armv7.tar.gz

# Make it executable
chmod +x picoclaw

# Clean up the archive
rm picoclaw_Linux_armv7.tar.gz
```

> `/media/fat/Scripts/` is MiSTer's standard scripts directory — a natural place for tools like PicoClaw.

> **Note:** The filename is case-sensitive (`Linux` with capital L). Check the
> [PicoClaw releases page](https://github.com/sipeed/picoclaw/releases) if the
> URL changes in a future version.

---

### Step 3 — Set up the workspace and run onboarding

Create a workspace directory for PicoClaw to operate in:

```bash
mkdir -p /media/fat/workspace
```

Then run the onboarding wizard:

```bash
./picoclaw onboard
```

This creates the configuration directory at `~/.picoclaw/` and walks you through initial setup.

---

### Step 4 — Configure your API keys

Edit the config file:

```bash
vi ~/.picoclaw/config.json
```

> MiSTer's minimal Linux may not include `nano`. Use `vi` instead, or install nano
> with your package manager if available.

#### Option A — OpenAI

```json
{
  "workspace": "/media/fat/workspace",
  "llm": {
    "provider": "openai",
    "api_key": "sk-YOUR_OPENAI_API_KEY",
    "model": "gpt-4o-mini"
  }
}
```

#### Option B — Anthropic (Claude)

```json
{
  "workspace": "/media/fat/workspace",
  "llm": {
    "provider": "anthropic",
    "api_key": "sk-ant-YOUR_ANTHROPIC_API_KEY",
    "model": "claude-haiku-4-5-20251001"
  }
}
```

> `claude-haiku-4-5-20251001` is recommended for MiSTer — it is fast, cheap, and capable.

#### Option C — LiteLLM (custom/local provider)

If you run a local LLM server (e.g., Ollama, LM Studio) or use LiteLLM as a proxy:

```json
{
  "workspace": "/media/fat/workspace",
  "llm": {
    "provider": "litellm",
    "base_url": "http://YOUR_SERVER_IP:4000",
    "api_key": "your-litellm-key-or-any-string",
    "model": "ollama/llama3"
  }
}
```

> You can combine multiple providers — PicoClaw supports fallback chains.
> The `workspace` path tells PicoClaw where it can read/write files on your MiSTer.

---

### Step 5 — Test it

```bash
cd /media/fat/Scripts
./picoclaw agent -m "What MiSTer FPGA cores are best for SNES games?"
```

You should get an AI response directly in your terminal. If this works, your setup is complete.

---

### Step 6 — Run as a system service (optional)

To have PicoClaw start automatically when MiSTer boots, create a startup script:

```bash
cat > /media/fat/Scripts/start_picoclaw.sh << 'EOF'
#!/bin/bash
cd /media/fat/Scripts
./picoclaw agent --daemon
EOF
chmod +x /media/fat/Scripts/start_picoclaw.sh
```

Then add it to MiSTer's startup by appending to `/media/fat/linux/user-startup.sh`:

```bash
echo '/media/fat/Scripts/start_picoclaw.sh &' >> /media/fat/linux/user-startup.sh
```

> **Note:** MiSTer uses a custom Linux image that may not have full `systemd` support.
> The `user-startup.sh` approach is the standard way to auto-start services on MiSTer.

Reboot to verify it starts automatically:

```bash
reboot
```

After reboot, SSH back in and check if PicoClaw is running:

```bash
ps aux | grep picoclaw
```

---

## Usage

### Ask a question from SSH

```bash
picoclaw agent -m "How do I configure the MiSTer video scaler?"
```

### Interactive chat mode

```bash
picoclaw chat
```

### Check logs

If running via `user-startup.sh`, redirect output to a log file by editing the startup script:

```bash
./picoclaw agent --daemon > /media/fat/Scripts/picoclaw.log 2>&1
```

Then tail the log:

```bash
tail -f /media/fat/Scripts/picoclaw.log
```

---

## Messaging Channels (optional)

PicoClaw supports connecting to messaging platforms so you can chat with your MiSTer AI from your phone:

| Platform | Setup |
|----------|-------|
| Telegram | Add your bot token to `config.json` |
| Discord | Add your Discord bot token |
| WhatsApp | Configure via the gateway settings |

See the [PicoClaw documentation](https://github.com/sipeed/picoclaw) for channel-specific setup.

---

## Troubleshooting

**`wget: command not found`**
MiSTer's minimal Linux does not include `wget`. The tutorial uses `curl` which is available by default.

**`exec format error`**
You downloaded the wrong architecture binary. Make sure you use `armv7` (not `arm64` or `amd64`).

**`picoclaw: not found` after reboot**
Add `/media/fat/Scripts` to your PATH, or always use the full path `/media/fat/Scripts/picoclaw`.

**API key errors**
Double-check your key in `~/.picoclaw/config.json`. Make sure there are no extra spaces or line breaks.

**No network on MiSTer**
Ensure your MiSTer is connected via Ethernet and has an IP address: `ip addr show`

---

## File Locations

| File | Path |
|------|------|
| PicoClaw binary | `/media/fat/Scripts/picoclaw` |
| Config file | `~/.picoclaw/config.json` |
| Workspace | `/media/fat/workspace` |
| Logs | `/media/fat/Scripts/picoclaw.log` (if configured) |
| Startup script | `/media/fat/Scripts/start_picoclaw.sh` |
| MiSTer user startup | `/media/fat/linux/user-startup.sh` |

---

## Credits

- **Tutorial author:** [@wolfshow](https://github.com/wolfshow) — first person in the world to install PicoClaw on MiSTer FPGA
- **PicoClaw:** [sipeed/picoclaw](https://github.com/sipeed/picoclaw) by Sipeed
- **MiSTer FPGA:** [MiSTer-devel](https://github.com/MiSTer-devel)

---

## License

This tutorial is released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) — do whatever you want with it.
