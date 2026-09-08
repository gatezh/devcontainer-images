# Docker maintenance cheatsheet (macOS)

Stock commands only — nothing custom to install. Background and the *why* live in
[`disk-usage.md`](./disk-usage.md).

---

## "Low disk space" warning — do this first

```bash
# 1. Real free space (df -h / reports the read-only system snapshot, ignore it)
df -h /System/Volumes/Data

# 2. How big is Docker really? (ls shows the sparse virtual max, du shows actual)
du -sh ~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw

# 3. What inside Docker is using it
docker system df
```

## Safe reclaim, in order of value

```bash
docker image prune -f            # dangling images only - safe, offsets daily :latest pulls
docker builder prune -f          # build cache beyond the keep-storage in daemon.json
docker container prune -f        # STOPPED containers only - check nothing was killed mid-work
```

`Docker.raw` auto-TRIMs after a prune, so the file shrinks on its own. No manual
`fstrim` needed on current Docker Desktop.

## The big one: the `vscode` volume

Usually the single largest item (23.5 GB when last measured). Nothing prunes it.

```bash
docker system df -v | grep -i vscode         # check its size first
```

**Closing VS Code is not enough, and neither is stopping the containers.**
`docker volume rm` blocks on any container that *references* the volume, running
or not — the containers must be removed. Expect this error otherwise:

```
Error response from daemon: remove vscode: volume is in use - [<ids>]
```

Full procedure:

```bash
# 1. Which containers reference it
docker ps -a --filter volume=vscode --format '{{.Names}}\t{{.State}}'

# 2. Stop any that are running. Note: devcontainers with
#    restart: unless-stopped come back by themselves and stay up even with
#    VS Code closed.
docker stop <names>

# 3. Remove them. NEVER add -v: that would delete the named volumes too,
#    including *-claude-config (credentials) and *-pgdata (databases).
docker rm <names>

# 4. Now the volume will go
docker volume rm vscode
```

Safe to do because devcontainer source is bind-mounted from the host and state
lives in named volumes, both of which survive `docker rm`. What you lose is the
container writable layer (mise/bun caches) — rebuilt on next "Reopen in
Container", along with a one-time VS Code server re-download.

Do this when the volume exceeds ~10 GB, roughly quarterly.

## VS Code's own cleanup (Command Palette)

- `Dev Containers: Clean Up Dev Containers…`
- `Dev Containers: Clean Up Dev Volumes…`

## Never run these

```bash
docker system prune -a --volumes    # DESTROYS pgdata / claude-config / fish-data
docker volume prune -a              # same - those volumes hold credentials and DBs
```

To release a *retired* project's volumes deliberately: `docker compose down -v`
in that project's folder.

---

## Reclaimed space didn't show up in `df`?

APFS local snapshots pin the freed blocks — copy-on-write means deleting data
inside `Docker.raw` returns nothing to the pool while a snapshot references it.

```bash
tmutil listlocalsnapshots /         # same-day snapshots are the usual cause
```

They expire on their own within ~24h and the space returns. To force it (this
is what macOS itself runs under pressure, `4` = urgency):

```bash
tmutil thinlocalsnapshots / 30000000000 4
```

Deleting local snapshots does **not** affect Time Machine backups on an external
or network disk.

---

## Docker is hung — every command just sits there

Symptom: `docker ps` never returns (rather than erroring). Usually means the
Linux VM died but the host-side backend still holds the socket.

```bash
# Confirm it
grep -iE 'no space|GET /error' \
  ~/Library/Containers/com.docker.docker/Data/log/host/com.docker.backend.log | tail

# Recover
osascript -e 'quit app "Docker Desktop"'
pgrep -f 'com\.docker\.back[e]nd'          # note the PID, then: kill -9 <pid>
open -a Docker
until docker info >/dev/null 2>&1; do sleep 5; done; echo "daemon up"
```

Note the `back[e]nd` bracket trick: a plain `pkill -f "com.docker.backend"`
matches the killing shell's own command line and kills itself instead.

**After a crash, `docker container prune -f` is not safe.** Containers that were
*running* are now `Exited (255)` and indistinguishable from long-idle ones.
Separate them by stop time before pruning:

```bash
docker inspect -f '{{.State.Status}}|{{.State.FinishedAt}}|{{.Name}}' $(docker ps -aq) | sort -t'|' -k2
```

Crash-killed containers all share the daemon-boot timestamp.

---

## Settings worth having

**Disk usage limit** — Docker Desktop → Settings → Resources → Advanced.
Set it *below* your typical free space (64 GB here). Without it the default is a
1 TB virtual disk, i.e. Docker can consume the whole Mac. Reducing the limit
recreates the disk and destroys all volumes — back up first.

**Log rotation** — `~/.docker/daemon.json`:

```json
{
  "builder": { "gc": { "enabled": true, "defaultKeepStorage": "5GB" } },
  "log-driver": "local"
}
```

`local` rotates and compresses; the default `json-file` grows unbounded.
Requires a Docker restart, and applies to newly created containers.

---

## Volume backups

Tarballs, manifest, and `backup.sh` / `restore.sh` live in
`~/docker-volume-backups/`. Restore verifies sha256 against `MANIFEST.txt` and
refuses to overwrite a volume that a running container has mounted.

```bash
~/docker-volume-backups/<date>/restore.sh --verify    # integrity check only
~/docker-volume-backups/<date>/restore.sh             # restore missing volumes
~/docker-volume-backups/<date>/restore.sh --force     # REPLACE existing ones
```

Worth backing up: `*-claude-config`, `*-fish-data`, `*-fish-history`, `*pgdata*`,
`*sql-data*`, `*storage-data*`.
Regenerable, don't bother: `vscode`, `*node-modules*`, `*playwright-browsers*`,
images, container layers.
