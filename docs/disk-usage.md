# Why Docker fills the disk, and what actually fixes it

Written after the 2026-09-07 incident: Docker Desktop's VM died with
`no space left on device`, taking every container with it. The Mac had 2.8 GB
free; `Docker.raw` had grown to 80 GB. This is the third occurrence.

The point of this document is that **almost none of it was a Docker bug or a
misconfiguration in this repo**. It is normal, expected growth in caches that
nothing prunes, behind a limit that was never set.

---

## The one setting that turns a nuisance into an outage

Docker Desktop's `settings-store.json` had **no `DiskSizeMiB` key**, so the VM
disk defaulted to a **1 TB** virtual maximum:

```
ls -lh ~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw
# -rw-r--r--  1 user  staff  1.0T   <- apparent (virtual max)
du -sh  ~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw
# 80G                                <- actually allocated (sparse file)
```

With a 1 TB ceiling on a 460 GB Mac, Docker's effective limit is *the entire
machine*. So instead of Docker hitting its own wall and returning a normal
`no space left on device` to whatever was writing, it kept growing until macOS
ran out — and the VM itself died.

**Fix: set a disk usage limit below your typical free space.** 64 GB was chosen
(steady-state need is ~25–30 GB). Docker Desktop → Settings → Resources →
Advanced → Disk usage limit.

This is the single highest-value change. It does not stop the growth; it makes
the failure survivable and local to Docker.

---

## Where the space actually goes

Measured on 2026-09-08, inside a 53 GB `Docker.raw`:

| Consumer | Size | Pruned by anything? |
|---|---|---|
| `vscode` volume (VS Code server + extension cache) | 23.5 GB | **No** |
| Named volumes (node_modules, pgdata, claude-config…) | 31 GB total | Only manually |
| Images | 9.5 GB | `docker image prune` |
| Build cache | 3.9 GB | builder GC (configured) |
| Container writable layers | 6.3 GB | `docker container prune` |

### 1. The `vscode` volume — the big one

Created automatically by the Dev Containers extension (nothing in this repo
mounts it) to cache the VS Code Server across container rebuilds. It contained:

- **11.1 GB of server installs** — 17 of them. One directory per VS Code commit,
  ~500 MB–1.3 GB each, going back ~9 months. Old ones are never deleted.
- **11 GB of `extensionsCache`** — of which `anthropic.claude-code` alone was
  **84 cached versions = 6.7 GB**, and `openai.chatgpt` 26 versions = 3.5 GB.
- **819 MB of orphaned `.vsix` downloads** — UUID-named temp files left behind by
  interrupted downloads.

Two things make this worse here than for most people:

- **AI extensions ship several releases per week.** A cache designed for
  monthly-ish updates now takes ~30x the churn, and keeps every version.
- **Running both Alpine and glibc devcontainers doubles it.** Every VS Code
  release is installed twice, as `linux-arm64` *and* `alpine-arm64`.

### 2. Daily `:latest` pulls leave dangling images

The images in this repo are rebuilt by CI daily, and each devcontainer's
`initializeCommand` runs `docker pull …:latest` on open. The previous ~2.5 GB
image becomes dangling locally and nothing removes it automatically — builder GC
only touches build cache, and the GHCR cleanup workflow only touches the remote
registry. Regular `docker image prune` is required.

### 3. node_modules volumes are duplicated per variant

Compose prefixes volume names with the project name, so the default and
`-sandbox` variants of a devcontainer get *separate* `node_modules` volumes even
though `${localWorkspaceFolderBasename}` was intended to share them. Opening both
variants stores dependencies twice.

### 4. Long-running containers accumulate writable layers

`mise install` / `bun install` caches land in the container's writable layer
rather than a volume. One long-lived container reached 13 GB on its own.

---

## Two macOS behaviours that make diagnosis confusing

**`df -h /` lies.** It reports the sealed read-only system snapshot. Real free
space is on the data volume:

```
df -h /System/Volumes/Data
```

**Freeing space inside Docker may not return it to the Mac.** APFS local Time
Machine snapshots are copy-on-write and pin the old blocks. After reclaiming
29 GB and watching `Docker.raw` shrink 80 → 51 GB, host free space did not move
at all, because two same-day snapshots still referenced the old contents. They
expire on their own within ~24h and the space then returns.

```
tmutil listlocalsnapshots /        # if space did not come back, look here first
```

---

## What to do about it — official tools only

Ranked by value:

1. **Set the Docker disk usage limit** (above). Do this regardless of everything
   else.
2. **Delete the `vscode` volume periodically** — stop your devcontainers, then
   `docker volume rm vscode`. The extension recreates it containing only the
   current server. Reclaims ~18 GB; costs one server re-download per
   devcontainer, once.
3. **`docker image prune -f`** on a regular basis — this is the one that offsets
   the daily `:latest` pulls.
4. **VS Code's own commands**: `Dev Containers: Clean Up Dev Containers…` and
   `Dev Containers: Clean Up Dev Volumes…`.

See [`docker-maintenance-cheatsheet.md`](./docker-maintenance-cheatsheet.md) for
the exact commands.

### There is no built-in cleanup to enable

Verified against Microsoft's documentation and the extension manifest: VS Code
ships **no** automatic pruning of the server cache or `extensionsCache`, and no
setting to bound their size. There is nothing being "missed".

The one relevant official switch is **`dev.containers.cacheVolume`** (default
`true`) — *"Controls whether a Docker volume should be used to cache the VS Code
server and extensions."* Setting it to `false` removes the shared volume
entirely, and the server then lives in each container's writable layer, which
`docker container prune` can reclaim.

**Not recommended for this setup**: because these devcontainers pull `:latest` on
every open and CI rebuilds daily, containers are recreated often, and each
recreation would re-download the server. The shared cache is genuinely earning
its keep here — it just needs occasional emptying.

---

## Things that look like solutions but are not

- **`docker system prune -a --volumes`** — wipes database and credential volumes
  (`*-pgdata`, `*-claude-config`, `*-fish-data`). Never run it here.
- **`docker volume prune`** — same problem; orphaned `*-claude-config` and
  `*-fish-data` volumes are kept deliberately, they hold credentials and history.
- **Scheduled-cleanup containers** — Watchtower was archived in Dec 2025 and
  `spotify/docker-gc` is unmaintained.
- **Shrinking the Docker disk to reclaim space** — reducing the limit recreates
  the disk image and destroys all volumes. Back up first (see the runbook in
  `~/docker-volume-backups/`).

---

## Prevention checklist

- [ ] Docker disk usage limit set (64 GB)
- [ ] `docker image prune -f` run periodically
- [ ] `vscode` volume emptied when it exceeds ~10 GB
- [ ] `"log-driver": "local"` in `~/.docker/daemon.json` — replaces unbounded
      `json-file` container logs with rotating, compressed ones
- [ ] Retired project? `docker compose down -v` in its folder releases its volumes
