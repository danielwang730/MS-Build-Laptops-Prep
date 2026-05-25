# MS Build 2026 — Laptop Prep Guide

**Event:** Microsoft Build 2026
**Dates:** June 2–3, 2026
**Hardware:** 50× AMD Ryzen AI Max 395

---

## Background

These 50 laptops were originally imaged for the **AMD AI DevDay (April 2026)** following the [ubuntu-setup](https://github.com/iswaryaalex/ubuntu-setup) guide. That process:

- Set up Windows + Ubuntu 24.04 dual boot (1 TB allocated to Ubuntu)
- Disabled Secure Boot and BitLocker
- Installed a mainline kernel (6.18.20), Docker, VS Code, and AMD ROCm 7.2.1
- Pulled a ROCm-enabled vLLM Docker image and installed Ollama
- Downloaded the `unsloth/gemma-4-E2B-it` (HuggingFace) and `qwen3:0.6b` (Ollama) models

The laptops were then used across multiple workshops at AMD AI DevDay. They need to be **reset and re-prepped** for the MS Build Space Blaster workshop, which uses **Lemonade Server**, **OpenCode**, and the **Qwen3.6-35B-A3B-ThinkingCoder** model.

The laptops should boot directly into Ubuntu 24.04. Credentials are:
```
Username: amd-user
Password: amd1234
```

---

## Part 1 — Reset the Previous Environment

### 1.1 Close open browser and terminal tabs

Manually close any open browser windows and terminal tabs before proceeding.

### 1.2 Stop any running Docker containers

```bash
# In a new terminal:
```

> If the terminal prompt shows a virtual environment (e.g. `(venv)` or `(base)`), deactivate it first:
> ```bash
> deactivate        # for Python venv
> conda deactivate  # for conda
> ```

```bash
# List all containers (running and stopped)
docker ps -a

# Stop and remove all containers
docker stop $(docker ps -aq) 2>/dev/null; docker rm $(docker ps -aq) 2>/dev/null
```

> If `docker ps -aq` returns nothing, there are no containers — that's fine.

### 1.3 Stop Ollama if it's serving a model

```bash
# Check if Ollama is running
systemctl is-active ollama

# If active, stop it
sudo systemctl stop ollama
```

---

## Part 2 — Install New Tools

### 2.1 Install Lemonade Server

Lemonade runs large AI models locally on the AMD GPU via ROCm.

```bash
sudo add-apt-repository ppa:lemonade-team/stable
sudo apt install lemonade-server
```

### 2.2 Install OpenCode

```bash
curl -fsSL https://opencode.ai/install | bash
source ~/.bashrc
```

### 2.3 Clone the workshop repo

```bash
cd ~
git clone https://github.com/iswaryaalex/space-battle-agentic-game-dev.git
cd space-battle-agentic-game-dev
```

### 2.4 Download the model

Start Lemonade and pull the model. **This step takes up to 15 minutes** — start it and let it run.

```bash
lemonade launch opencode
```

When prompted to select a recipe, choose:

```
8) Qwen3.6-35B-A3B-ThinkingCoder.json
```

Lemonade will download and cache the model, then launch OpenCode.

---

## Part 3 — Verify Everything Works

### 3.1 Confirm agents are loaded

Inside the OpenCode TUI (which should have launched in the previous step), type:

```
/agents
```

You should see (not necessarily in this order — use the arrow keys to scroll through the list):

```
ui-renderer
physics-movement
gameplay-rules
vfx-polish      (optional)
boss-ai         (optional)
```

> You may also see a `build-native` agent — this is a built-in OpenCode agent, not part of the workshop. You can ignore it.

If all five agents appear, type `hi` into the chat and wait a few seconds. If the model responds, the laptop is ready.

### 3.2 Shut down OpenCode and Lemonade

After confirming the agents, exit OpenCode — press `q` or `Ctrl+C` inside the TUI — then **restart the laptop**. A reboot clears GPU memory even if the Lemonade service is still running in the background.

The participant will launch `lemonade launch opencode` themselves at the start of the workshop.

### 3.3 Quick sanity checks

Open a new terminal and run:

```bash
# Confirm ROCm can see the GPU
rocminfo | grep -E "Name:|Marketing Name:" | head -6

# Confirm Lemonade service is running
systemctl status lemond | head -9

# Confirm GPU is idle
rocm-smi
```

> As long as these don't error out and you see a response, you're good. On `rocm-smi`, confirm VRAM usage is below 2% before handing off the laptop.

### 3.4 Final cleanup

1. Close all open terminal windows.
2. Open the browser, then close it. Even if it was already closed after the reboot, open a fresh window and close it — this clears any cached session data from the previous user.

---

## Troubleshooting

**`docker stop` / `docker rm` errors with empty list**
This just means no containers exist — safe to ignore.

**Ollama not found / not active**
Ollama may not be running at all — that's fine, no action needed.

**Wrong model selected in Lemonade**
Exit OpenCode (`Ctrl+C`), then re-run `lemonade launch opencode` and select option 8.

**GPU not detected by ROCm**
- Confirm Secure Boot is still disabled in BIOS (F10 to enter)
- Reboot and try again
- Run `rocminfo` and `rocm-smi` to diagnose

**Lemonade service needs to be manually restarted (serious errors only)**
You should not need to do this under normal circumstances — a reboot is sufficient. Only use this if you are seeing a serious error with the Lemonade service itself:
```bash
sudo systemctl stop lemond
systemctl start lemond
```

---

## Summary Checklist

- [ ] Browser and terminal tabs closed
- [ ] Docker containers stopped and removed
- [ ] Ollama stopped (if it was running)
- [ ] Lemonade Server installed
- [ ] OpenCode installed
- [ ] Workshop repo cloned to `~/space-battle-agentic-game-dev`
- [ ] `Qwen3.6-35B-A3B-ThinkingCoder` model downloaded via `lemonade launch opencode`
- [ ] `/agents` inside OpenCode shows all 3 required agents
