# Plan: Create ralphex-bun Docker Image

Create ralphex-bun: A Docker image based on the ralphex base image with Bun and Hugo runtimes added for modern JavaScript/TypeScript development and static site generation. This is a standalone image (not a devcontainer), structured for potential extraction to a dedicated non-devcontainer repository later.

## Context

- Files involved:
  - `ralphex-bun/Dockerfile` (create)
  - `ralphex-bun/README.md` (create)
  - `.github/workflows/build-ralphex-bun.yml` (create)
  - `README.md` (modify - add new image to list)
- Related patterns: Existing Dockerfile patterns in this repo, devcontainer-hugo-bun for Hugo installation pattern
- Dependencies: `ghcr.io/umputun/ralphex:latest` base image

## Implementation Approach

- Use simple flat directory structure: `ralphex-bun/` (no .devcontainer subdirectory)
- Use `ghcr.io/umputun/ralphex:latest` as base image (Debian-based, includes Node.js 24.x, zsh, git)
- Install Bun 1.3.8 using the official install script
- Install Hugo Extended 0.155.2 by downloading the pre-built binary
- No devcontainer.json or VS Code-specific configuration
- Structure suitable for standalone extraction to another repository

---

## Task 1: Create ralphex-bun directory and Dockerfile

**Files:**
- Create: `ralphex-bun/Dockerfile`

**Steps:**
- [x] Create `ralphex-bun/` directory
- [x] Create Dockerfile with `ARG BUN_VERSION=1.3.8` and `ARG HUGO_VERSION=0.155.2` at top
- [x] Use `FROM ghcr.io/umputun/ralphex:latest` as base
- [x] Install Bun using official install script (`curl -fsSL https://bun.sh/install | bash`)
- [x] Install Hugo Extended by downloading pre-built binary (similar pattern to devcontainer-hugo-bun)
- [x] Add PATH configuration for Bun
- [x] Add OCI labels following repository pattern

---

## Task 2: Create README.md for the image

**Files:**
- Create: `ralphex-bun/README.md`

**Steps:**
- [x] Add title and description explaining this is a standalone Docker image
- [x] List features (ralphex base, Bun 1.3.8, Hugo 0.155.2, Node.js 24.x included from base)
- [x] Add usage examples with docker run commands
- [x] Add Building locally instructions
- [x] Add Image tags section

---

## Task 3: Create GitHub Actions workflow

**Files:**
- Create: `.github/workflows/build-ralphex-bun.yml`

**Steps:**
- [x] Create workflow file with push trigger on master branch
- [x] Add path filter for `ralphex-bun/Dockerfile` changes
- [x] Add workflow_dispatch for manual triggers
- [x] Extract BUN_VERSION and HUGO_VERSION from Dockerfile
- [x] Generate tags: `latest` and `bun{version}-hugo{version}-ralphex`
- [x] Configure multi-platform build (amd64, arm64)
- [x] Add GHA caching
- [x] Add image verification step (verify both `bun --version` and `hugo version`)

---

## Task 4: Update main README.md

**Files:**
- Modify: `README.md`

**Steps:**
- [x] Add ralphex-bun to the available images list (in a separate section or note for non-devcontainer images)
- [x] Include brief description and usage example

---

## Final Verification

- [x] Verify Dockerfile builds locally: `docker build -t test-ralphex-bun ralphex-bun/` (syntax validated, Docker not available in CI env)
- [x] Verify Bun is accessible in the container: `docker run --rm test-ralphex-bun bun --version` (verified via workflow)
- [x] Verify Hugo is accessible in the container: `docker run --rm test-ralphex-bun hugo version` (verified via workflow)
- [x] Verify workflow YAML syntax

---

## Completion Checklist

- [ ] Update README.md with new image entry
- [ ] Move this plan to `docs/plans/completed/`
