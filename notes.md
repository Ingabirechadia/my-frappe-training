# frappe_docker setup notes

Log of every problem hit while setting up the dev environment, and how it was solved.
Keep entries short: exact error, cause, fix. Append as you go.

---

## Problem: Dev Containers failed with npipe/docker API connection error
**When:** Day 2, first attempt to run `Dev Containers: Reopen in Container`
**Error:**
```
failed to connect to the docker API at npipe:////./pipe/dockerDesktopLinuxEngine;
check if the path is correct and if the daemon is running: open //./pipe/dockerDesktopLinuxEngine:
The system cannot find the file specified.
```
**Cause:** Docker Desktop application wasn't fully running yet. VS Code can auto-launch
Docker Desktop, but that launch is asynchronous — it doesn't wait for the engine to
actually be ready before trying to connect, so there's a gap where this fails.
**Fix:** Always open Docker Desktop manually first and wait for the "Engine running"
status (green, bottom-left of the Docker Desktop window) *before* triggering
`Dev Containers: Reopen in Container` in VS Code.

---

## Problem: Wrong devcontainer.json picked during setup
**When:** Day 2, when VS Code prompted "Select a devcontainer.json file"
**Cause:** Two options appeared — the live copy under `.devcontainer/devcontainer.json`
(created via `cp -R devcontainer-example .devcontainer`) and the original template
under `devcontainer-example/devcontainer.json`.
**Fix:** Always select the one under `.devcontainer\devcontainer.json` — that's the
working copy VS Code should build from, not the source template.

---

## Problem: docker compose up failed on a Wayland socket mount
**When:** Day 2, container build got much further (image pulled, network/volume created)
but failed on the last step
**Error:**
```
Error response from daemon: accessing specified distro mount service:
stat /run/guest-services/distro-services/ubuntu.sock: no such file or directory
```
**Cause:** The Dev Containers extension automatically tries to bind-mount the WSL
distro's Wayland socket (`\\wsl.localhost\Ubuntu\mnt\wslg\runtime-dir\wayland-0`) to
support GUI app forwarding — not something needed for this project. Docker Desktop's
WSL guest-mount service for the Ubuntu distro wasn't exposing that socket, so the whole
`docker compose up` failed on that one mount.
Note: setting `dev.containers.gpuAvailability` to `"none"` did NOT stop this mount
attempt — it's a separate feature from GPU detection.
**Fix:** Disable the Wayland mount specifically — VS Code Settings (`Ctrl+,`) → search
`wayland` → uncheck **Dev › Containers: Mount Wayland Socket**
(`dev.containers.mountWaylandSocket: false`).

---

## Problem: Setting couldn't be applied — "cannot be applied in this window"
**When:** Day 2, trying to change the Wayland socket setting while still in a
disconnected remote/container window
**Cause:** `dev.containers.mountWaylandSocket` is a local/host-only setting. VS Code
was still in a remote context (even though disconnected), so it refused to apply a
host-side setting there.
**Fix:** Return to a fully local window first (`Dev Containers: Reopen Folder Locally`,
or close and reopen VS Code), confirm the title bar has no `[Dev Container]` suffix,
*then* change the setting, *then* retry `Reopen in Container`.

---


