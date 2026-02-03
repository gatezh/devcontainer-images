# Devcontainer Images

This repository contains Dockerfiles for custom Docker images hosted on GitHub Container Registry (ghcr.io).

## 📚 Image Documentation

### Devcontainer Images

- **[devcontainer-bun](./devcontainer-bun/README.md)** - Bun development container
- **[devcontainer-claude-bun](./devcontainer-claude-bun/README.md)** - Claude Code development container with firewall sandbox
- **[devcontainer-hugo-bun](./devcontainer-hugo-bun/README.md)** - Hugo Extended + Bun development container

### Standalone Docker Images

- **[ralphex-bun](./ralphex-bun/README.md)** - Bun + Hugo Extended on ralphex base (standalone image)

## Repository Structure

Each subdirectory represents a Docker image project with the following structure:

```
image-name/
├── .devcontainer/
│   ├── Dockerfile          # Source of truth for the image
│   └── devcontainer.json   # Dev container configuration (uses "build")
└── ...
```

## Available Images

### devcontainer-bun

Bun development container for modern JavaScript/TypeScript development.

**Usage in other projects:**

```json
{
  "image": "ghcr.io/<username>/devcontainer-bun:latest"
}
```

### devcontainer-claude-bun

Claude Code development container with Bun runtime, Claude Code CLI, and a restrictive firewall sandbox.

**Usage in other projects:**

```json
{
  "image": "ghcr.io/<username>/devcontainer-claude-bun:latest",
  "runArgs": ["--cap-add=NET_ADMIN", "--cap-add=NET_RAW"],
  "postStartCommand": "sudo /usr/local/bin/init-firewall.sh"
}
```

### devcontainer-hugo-bun

Hugo development container with Bun runtime.

**Usage in other projects:**

```json
{
  "image": "ghcr.io/<username>/devcontainer-hugo-bun:latest"
}
```

### ralphex-bun

Standalone Docker image based on ralphex with Bun 1.3.8 and Hugo Extended 0.155.2 for modern JavaScript/TypeScript development and static site generation.

**Usage:**

```bash
# Pull and run interactively
docker pull ghcr.io/<username>/ralphex-bun:latest
docker run -it --rm -v $(pwd):/workspace -w /workspace ghcr.io/<username>/ralphex-bun:latest

# Run Bun commands
docker run --rm -v $(pwd):/workspace -w /workspace ghcr.io/<username>/ralphex-bun:latest bun run index.ts

# Run Hugo commands
docker run --rm -v $(pwd):/workspace -w /workspace -p 1313:1313 ghcr.io/<username>/ralphex-bun:latest hugo server --bind 0.0.0.0
```

## Adding a New Image

1. Create a new directory with your image name
2. Add `.devcontainer/Dockerfile` with your image definition
3. Add `.devcontainer/devcontainer.json` that references the Dockerfile
4. Build and push the image to GitHub Container Registry
5. Update this README with usage instructions

## Building and Publishing

Images from this repository are built and published to GitHub Container Registry. Other projects can reference these images in their `devcontainer.json` files using the `"image"` property.
