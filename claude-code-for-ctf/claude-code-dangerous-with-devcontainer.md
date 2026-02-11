https://chatgpt.com/c/698cf24c-7a04-838f-b40f-a90a730dc5a2

# Claude Code YOLO Mode in Anthropic Devcontainer (Windows 11 + WSL2)

## 0) One-time prerequisites (Windows side)

### Install Docker Desktop (Windows)
1. Install **Docker Desktop**.
2. Open **Docker Desktop → Settings → General**
   - Ensure it uses the **WSL 2 backend** (usually default on Windows 11).

### Enable Docker WSL integration
1. Docker Desktop → **Settings → Resources → WSL Integration**
2. Turn **ON** your distro (e.g., **Ubuntu**).

### Install VS Code (Windows)
1. Install **Visual Studio Code**.

### Install VS Code extension
1. In VS Code, install **Dev Containers** extension (Microsoft).

---

## 1) Quick-start: run the official Claude Code devcontainer once (sanity check)

### In WSL Ubuntu terminal
```bash
cd ~
git clone https://github.com/anthropics/claude-code.git
cd claude-code
code .
```

3) Use the devcontainer for YOUR CTF repo (recommended workflow)
3.1 Copy the devcontainer into your CTF project

Example (replace with your repo path):

cd ~/ctf/my-challenge
mkdir -p .devcontainer


Copy Anthropic’s reference devcontainer into your project:

cp -r ~/claude-code/.devcontainer/* .devcontainer/


Open your project in VS Code:

code .

3.2 Reopen your project in the container

In VS Code:

Ctrl+Shift+P → Dev Containers: Reopen in Container

===

4) Add “CTF power tools” (gdb/rr/rev) to the container
4.1 Edit .devcontainer/Dockerfile and add packages

Open .devcontainer/Dockerfile and add to the apt-get install list (or add another RUN).

Common CTF packages

pwn/debug: gdb, gdb-multiarch, rr, strace, ltrace, patchelf, elfutils, binutils, file

rev: radare2, unzip, p7zip-full

python tooling: python3, python3-pip, python3-venv

build basics: build-essential, pkg-config, cmake

Example snippet (add as a new RUN near the top, before cleanup):

RUN apt-get update && apt-get install -y --no-install-recommends \
  gdb gdb-multiarch rr strace ltrace patchelf elfutils binutils file \
  python3 python3-pip python3-venv \
  build-essential pkg-config cmake \
  p7zip-full \
  && apt-get clean && rm -rf /var/lib/apt/lists/*


4.2 Rebuild the container

In VS Code:

Ctrl+Shift+P → Dev Containers: Rebuild Container

====

5) Make gdb + rr work well inside containers (important)
5.1 Enable ptrace capabilities in devcontainer

Edit .devcontainer/devcontainer.json and add:

"runArgs": [
  "--cap-add=SYS_PTRACE",
  "--security-opt=seccomp=unconfined"
]


5.2 If rr fails on WSL2 (last resort)

rr can be picky about kernel/perf settings. If it still fails, try:

"runArgs": [
  "--privileged"
]

Use --privileged only if you must (it weakens isolation).

6) Keep Claude auth + config persistent across rebuilds (avoid re-login)

Edit .devcontainer/devcontainer.json and ensure you have a mount like:

"mounts": [
  "source=claude-code-config-${devcontainerId},target=/home/node/.claude,type=volume"
]

7) Firewall notes (why tools may not “phone home”)

Anthropic’s devcontainer uses a default-deny outbound firewall (only whitelists specific destinations).
If you need more internet access (apt/pip/CTF downloads), edit:

.devcontainer/init-firewall.sh

Whitelist what you need, otherwise downloads/requests may fail by design.

8) Finally: run YOLO mode for the actual CTF work

Inside the devcontainer terminal in your challenge repo:

claude --dangerously-skip-permissions


