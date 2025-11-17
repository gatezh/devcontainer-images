# Build devcontainer-hugo-bun Workflow

## Overview

Automatically builds and pushes multiplatform Docker images for `devcontainer-hugo-bun` to GitHub Container Registry.

## How It Works

### Triggers

The workflow runs when:
- **Any push** that changes `devcontainer-hugo-bun/.devcontainer/Dockerfile`
- **Manual trigger** via Actions tab → "Build devcontainer-hugo-bun" → Run workflow

### What It Does

1. **Extracts versions** from Dockerfile ARGs:
   ```dockerfile
   ARG HUGO_VERSION=0.152.2
   ARG BUN_VERSION=1.3.2
   ```

2. **Generates tags**:
   - `ghcr.io/OWNER/devcontainer-hugo-bun:latest`
   - `ghcr.io/OWNER/devcontainer-hugo-bun:hugo0.152.2-bun1.3.2-alpine`

3. **Builds** for both `linux/amd64` and `linux/arm64`

4. **Pushes** to GitHub Container Registry

5. **Verifies** both platform images can be pulled

## Usage

### Automatic Build

Just update the Dockerfile and push:

```bash
# Edit devcontainer-hugo-bun/.devcontainer/Dockerfile
# Change: ARG HUGO_VERSION=0.153.0

git add devcontainer-hugo-bun/.devcontainer/Dockerfile
git commit -m "Update Hugo to 0.153.0"
git push
```

The workflow automatically builds and tags as `hugo0.153.0-bun1.3.2-alpine`.

### Manual Build

1. Go to **Actions** tab
2. Select **Build devcontainer-hugo-bun**
3. Click **Run workflow**
4. Select branch and click **Run workflow**

## Requirements

- Repository must have `packages: write` permission (automatically granted to `GITHUB_TOKEN`)
- First time: package visibility may need to be set to public in repository settings

## Result

Images available at:
```
ghcr.io/OWNER/devcontainer-hugo-bun:latest
ghcr.io/OWNER/devcontainer-hugo-bun:hugo{VERSION}-bun{VERSION}-alpine
```

## Troubleshooting

**Workflow doesn't trigger?**
- Ensure you're pushing changes to the Dockerfile path
- Check Actions tab for workflow runs

**Can't pull images?**
- Check package visibility in repository settings
- Verify the workflow completed successfully
- Ensure you're logged in: `docker login ghcr.io`
