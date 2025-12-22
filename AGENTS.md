# AI Agent Instructions

This document defines patterns, conventions, and guidelines for AI agents working with this repository.

## Repository Overview

This repository contains Dockerfiles for custom devcontainer images hosted on GitHub Container Registry (ghcr.io). Each image is designed for VS Code Dev Containers with specific development environments.

## Directory Structure

```
repository-root/
├── AGENTS.md                    # AI agent instructions (this file)
├── CLAUDE.md                    # Symlink to AGENTS.md
├── README.md                    # Repository documentation
├── .github/
│   └── workflows/
│       ├── README.md            # Workflow documentation
│       └── build-*.yml          # GitHub Actions workflows
└── devcontainer-{name}/
    ├── README.md                # Image-specific documentation
    └── .devcontainer/
        ├── Dockerfile           # Image definition (source of truth)
        ├── devcontainer.json    # VS Code devcontainer configuration
        └── *.sh                 # Optional scripts (e.g., init-firewall.sh)
```

## Naming Conventions

### Image Names
- Format: `devcontainer-{primary-tool}` or `devcontainer-{primary-tool}-{secondary-tool}`
- Examples: `devcontainer-bun`, `devcontainer-hugo-bun`, `devcontainer-claude-bun`

### Version ARGs in Dockerfile
- Place at the top of Dockerfile
- Format: `ARG {TOOL}_VERSION={version}`
- Examples:
  ```dockerfile
  ARG BUN_VERSION=1.3.5
  ARG HUGO_VERSION=0.152.2
  ARG CLAUDE_CODE_VERSION=latest
  ```

### Image Tags
- Always include `latest` tag
- Version-specific tag format: `{tool}{version}-{variant}`
- Examples:
  - `ghcr.io/owner/devcontainer-bun:latest`
  - `ghcr.io/owner/devcontainer-bun:bun1.3.5-alpine`
  - `ghcr.io/owner/devcontainer-claude-bun:bun1.3.5-slim`

## Dockerfile Patterns

### Required OCI Labels
All Dockerfiles must include these labels at the end:

```dockerfile
LABEL org.opencontainers.image.source="https://github.com/{owner}/devcontainer-images"
LABEL org.opencontainers.image.description="{Brief description}"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.title="{image-name}"
LABEL org.opencontainers.image.url="https://github.com/{owner}/devcontainer-images"
```

### Base Images
- Prefer official images from Docker Hub
- Use slim/alpine variants when possible
- Bun: `oven/bun:{version}-alpine` or `oven/bun:{version}-slim`
- Node: `node:{version}` or `node:{version}-slim`

### Common Packages
Minimal images should include:
- `ca-certificates` - HTTPS connections
- `git` - Version control
- `zsh` - Better shell for VS Code integration

## devcontainer.json Patterns

### File Header
```jsonc
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: {relevant-reference-url}
```

### node_modules Mount
Always include to keep node_modules out of host machine:
```jsonc
"mounts": [
  // Keep node_modules out of a host machine
  "source=${localWorkspaceFolderBasename}-node_modules,target=${containerWorkspaceFolder}/node_modules,type=volume"
]
```

### VS Code Extensions
- Group extensions by category with header comments
- Include description comment for each extension
- Format:
```jsonc
"extensions": [
  // **Category Name**
  // Extension Description
  "publisher.extension-id",

  // **Another Category**
  // Another Extension Description
  "another.extension"
]
```

### Common Extension Categories
- `**Claude Code**` - AI assistant
- `**Bun**` - Bun runtime support
- `**Code Quality**` - Biome (formatter and linter)
- `**Git**` - GitLens
- `**Tailwind**` - Tailwind CSS tooling
- `**Hugo**` - Hugo static site generator (when applicable)

### Required VS Code Settings
```jsonc
"settings": {
  "terminal.integrated.defaultProfile.linux": "zsh",
  // Suppress extension recommendation prompts
  "extensions.ignoreRecommendations": true
}
```

## GitHub Actions Workflow Patterns

### Trigger Configuration
```yaml
on:
  push:
    branches:
      - master
    paths:
      - '{image-name}/.devcontainer/Dockerfile'
      - '{image-name}/.devcontainer/*.sh'  # If scripts exist
  workflow_dispatch:
```

### Version Extraction
Extract versions from Dockerfile ARGs:
```yaml
- name: Extract versions from Dockerfile
  id: versions
  run: |
    DOCKERFILE="{image-name}/.devcontainer/Dockerfile"
    VERSION=$(grep '^ARG {TOOL}_VERSION=' "$DOCKERFILE" | cut -d'=' -f2)
    echo "{tool}=$VERSION" >> $GITHUB_OUTPUT
```

### Multi-platform Build
Always build for both architectures:
```yaml
platforms: linux/amd64,linux/arm64
```

### Caching
Use GitHub Actions cache:
```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

## README Patterns

### Image README Structure
1. Title and brief description
2. Features list
3. Quick Start (pre-built image usage)
4. Building locally instructions
5. Configuration details
6. Image tags
7. Customization options
8. Resources/links

### Main README Structure
1. Repository overview
2. Image documentation links
3. Repository structure
4. Available images with usage examples
5. Adding a new image guide
6. Building and publishing info

## When Adding a New Image

1. Create directory: `devcontainer-{name}/`
2. Add `.devcontainer/Dockerfile` following patterns above
3. Add `.devcontainer/devcontainer.json` following patterns above
4. Add `README.md` with image documentation
5. Create `.github/workflows/build-devcontainer-{name}.yml`
6. Update main `README.md` with new image entry

## Code Style

- Use 2-space indentation in JSON/YAML files
- Use comments liberally in devcontainer.json (JSONC)
- Keep Dockerfile instructions organized logically:
  1. ARG declarations
  2. FROM statement
  3. Package installation
  4. User/permission setup
  5. Tool installation
  6. Labels (at the end)
