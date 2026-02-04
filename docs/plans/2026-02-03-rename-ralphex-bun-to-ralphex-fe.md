# Rename ralphex-bun image to ralphex-fe

This plan renames the standalone Docker image from "ralphex-bun" to "ralphex-fe" across all files and references.

## Context

- Files involved:
  - ralphex-bun/ directory (rename to ralphex-fe/)
  - ralphex-bun/Dockerfile (update labels)
  - ralphex-bun/README.md (update all references)
  - .github/workflows/build-ralphex-bun.yml (rename and update contents)
  - README.md (update references in root readme)
- Related patterns: Existing naming conventions in the repository
- Dependencies: None

## Implementation Approach

- Simple find-and-replace approach for text changes
- Directory and file renames handled separately
- No code logic changes, purely renaming

## Task 1: Rename directory and workflow file

**Files:**
- Rename: `ralphex-bun/` -> `ralphex-fe/`
- Rename: `.github/workflows/build-ralphex-bun.yml` -> `.github/workflows/build-ralphex-fe.yml`

**Steps:**
- [x] Rename ralphex-bun directory to ralphex-fe
- [x] Rename build-ralphex-bun.yml to build-ralphex-fe.yml

## Task 2: Update Dockerfile labels

**Files:**
- Modify: `ralphex-fe/Dockerfile`

**Steps:**
- [x] Update image title label from "ralphex-bun" to "ralphex-fe"
- [x] Update image description to replace "-bun" with "-fe"

## Task 3: Update README inside ralphex-fe directory

**Files:**
- Modify: `ralphex-fe/README.md`

**Steps:**
- [x] Update title from "ralphex-bun" to "ralphex-fe"
- [x] Update all docker image references from ralphex-bun to ralphex-fe
- [x] Update all docker build/run examples
- [x] Update version tag format references

## Task 4: Update GitHub Actions workflow

**Files:**
- Modify: `.github/workflows/build-ralphex-fe.yml`

**Steps:**
- [x] Update workflow name from "Build ralphex-bun" to "Build ralphex-fe"
- [x] Update IMAGE_NAME env variable from "ralphex-bun" to "ralphex-fe"
- [x] Update paths trigger from ralphex-bun/ to ralphex-fe/
- [x] Update DOCKERFILE path from ralphex-bun/ to ralphex-fe/
- [x] Update context and file paths in build step

## Task 5: Update root README.md

**Files:**
- Modify: `README.md`

**Steps:**
- [ ] Update ralphex-bun references to ralphex-fe in documentation links section
- [ ] Update usage examples with new image name
- [ ] Update repository structure examples if applicable

## Verification

- [ ] Verify all file renames completed
- [ ] Search codebase for any remaining "ralphex-bun" references
- [ ] Verify workflow file syntax is valid (yaml lint)

## Completion

- [ ] Move this plan to `docs/plans/completed/`
