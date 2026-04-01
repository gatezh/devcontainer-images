# Adding a New Image

## Devcontainer Image

1. Create `{name}/.devcontainer/Dockerfile`
2. Create `{name}/.devcontainer/devcontainer.json`
3. Create `{name}/README.md`
4. Create `.github/workflows/build-{name}.yml`
5. Update root `README.md` with new image entry

## Standalone Docker Image

1. Create `{name}/Dockerfile` (no `.devcontainer/` subdirectory)
2. Create `{name}/README.md`
3. Create `.github/workflows/build-{name}.yml`
4. Update root `README.md` with new image entry
