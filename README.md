# Devcontainer Images

This repository contains Dockerfiles for custom Docker images hosted on GitHub Container Registry (ghcr.io).

## 📚 Image Documentation

- **[devcontainer-bun](./devcontainer-bun/README.md)** - Bun development container
- **[devcontainer-claude](./devcontainer-claude/README.md)** - Claude Code development container with firewall sandbox
- **[devcontainer-hugo-bun](./devcontainer-hugo-bun/README.md)** - Hugo Extended + Bun development container

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

### devcontainer-claude

Claude Code development container with Node.js 20, Claude Code CLI, and a restrictive firewall sandbox.

**Usage in other projects:**

```json
{
  "image": "ghcr.io/<username>/devcontainer-claude:latest",
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

## Adding a New Image

1. Create a new directory with your image name
2. Add `.devcontainer/Dockerfile` with your image definition
3. Add `.devcontainer/devcontainer.json` that references the Dockerfile
4. Build and push the image to GitHub Container Registry
5. Update this README with usage instructions

## Building and Publishing

Images from this repository are built and published to GitHub Container Registry. Other projects can reference these images in their `devcontainer.json` files using the `"image"` property.
