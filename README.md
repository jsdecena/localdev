# Isolated Dev Container

Use **Cursor on your Mac**, but keep **project files and git SSH keys inside Docker**—not in `~/code`, `~/Downloads`, or your Mac’s `~/.ssh`. Run **Agent** in this window so it works on `/workspace` in a container, not on your home folder.

Background: agents can follow malicious instruction files dropped in places like [Downloads](https://forever.security/blog/hijacking-ai-agents-via-downloads-folder/). This template is one way to narrow what the agent can reach. It is not perfect—still use approvals/sandboxing and do not run Agent in other windows opened on local Mac folders.

---

## Setup (do these steps in order)

### 1. Install and start Docker

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/).
2. Open Docker Desktop and wait until it is running (whale icon steady).
3. In **Terminal.app**, run `docker ps`. It should print a table (even if empty). If not, fix Docker before continuing.

### 2. Install Dev Containers in Cursor

1. Open [Cursor](https://cursor.com/).
2. Press **`Cmd+Shift+X`** → search **`Dev Containers`**.
3. Install **Dev Containers** by **Anysphere** (`anysphere.remote-containers`).
4. Reload Cursor if it asks.

Check: **`Cmd+Shift+P`** → type **`Dev Containers`** → you should see **Dev Containers: Reopen in Container**.

### 3. Open this template in the container

1. Clone this repo to your Mac (any folder is fine—it is only config):

   ```bash
   git clone https://github.com/YOUR_USER/YOUR_REPO.git
   cd YOUR_REPO
   ```

2. In Cursor: **File → Open Folder** → choose that cloned folder.
3. **`Cmd+Shift+P`** → **Dev Containers: Reopen in Container**.
4. Wait until the build finishes (first time can take **5–15 minutes**). The terminal prompt should look like `vscode@…:/workspace$`.

You are now inside the container. **`/workspace` is empty** on first run. Your app code does **not** live in the template folder on your Mac—it goes in `/workspace` below.

### 4. One-time: SSH key for GitHub

Run these **inside the container terminal** (only once; keys are saved on Docker volume `localdev-ssh` and survive rebuild):

```bash
ssh-keygen -t ed25519 -a 100 -C "you@devcontainer" -f ~/.ssh/id_ed25519 -N ""
cat ~/.ssh/id_ed25519.pub
```

1. Copy the printed key.
2. GitHub → **Settings → SSH and GPG keys → New SSH key** → paste → save.
3. Test:

   ```bash
   ssh -T git@github.com
   ```

### 5. Clone your project

Still **inside the container**:

```bash
cd /workspace
git clone git@github.com:YOUR_ORG/your-app.git
cd your-app
```

In Cursor: **File → Open Folder** → `/workspace/your-app` (so the editor root is your app).

### 6. Run the app

```bash
cd /workspace/your-app
nvm install && nvm use    # skip if you do not use Node / no .nvmrc
yarn install
yarn dev
```

Open the URL from the terminal, or use Cursor’s **Ports** panel (`localhost:…` on your Mac). If the browser cannot connect, configure your dev server to listen on `0.0.0.0` (see your framework’s docs).

**Docker Compose** (if the repo has it):

```bash
cd /workspace/your-app
docker compose up -d
```

---

## Daily use

| Goal | Action |
| --- | --- |
| Start working | Open this template folder in Cursor → **Reopen in Container** (or reopen the `/workspace/your-app` window if you saved it) |
| Change Node version | `nvm install && nvm use` in the project folder |
| Rebuild after editing `.devcontainer/devcontainer.json` | **Dev Containers: Rebuild Container** |

Use **Agent only in the dev-container window**, not in a separate Cursor window opened on `~/Downloads` or `~/code`.

---

## If something fails

| Problem | Fix |
| --- | --- |
| No **Dev Containers** commands | Install `anysphere.remote-containers`, reload Cursor |
| “Cannot connect to the Docker daemon” | Start Docker Desktop, restart Cursor |
| `Permission denied` on `git clone` into `/workspace` | **Rebuild Container** (runs permission fix), or run `sudo chown -R vscode:vscode /workspace` once |
| `yarn: command not found` | Run `corepack enable`, or **Rebuild Container** |
| `docker: command not found` | **Rebuild Container** |
| First container create is slow | Normal—pulling image and installing Node + Docker-in-Docker |

---

## Reset (optional)

Run on your **Mac** in Terminal.app (stops using data in that volume):

```bash
docker volume rm localdev-workspace   # deletes all projects in /workspace
docker volume rm localdev-ssh         # deletes container SSH keys; re-add key on GitHub
```

Then **Reopen in Container** and repeat setup from step 4.

---

## What this repo contains

Only **`.devcontainer/devcontainer.json`** (Ubuntu base image, nvm, Docker-in-Docker, two named volumes). No app code, no Dockerfile.
