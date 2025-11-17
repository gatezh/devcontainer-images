# GitHub Actions Workflows

## Multiplatform Image Build Workflow

### Overview

The `build-multiplatform.yml` workflow automatically builds and pushes multiplatform Docker images to GitHub Container Registry (ghcr.io) whenever a Dockerfile is updated in any image subdirectory.

### Features

- ✅ **Automatic triggering** when Dockerfiles are modified
- ✅ **Multiplatform builds** (linux/amd64, linux/arm64)
- ✅ **Dynamic version tagging** extracted from Dockerfile ARGs
- ✅ **Manual workflow dispatch** for on-demand builds
- ✅ **Build caching** for faster builds
- ✅ **Automatic image verification** after push

### How It Works

#### 1. Trigger Conditions

The workflow runs when:
- **Push to main branch** with changes to any `Dockerfile` or `*/.devcontainer/Dockerfile`
- **Manual trigger** via workflow_dispatch (specify image name)

#### 2. Change Detection

- Automatically detects which image directory contains the modified Dockerfile
- Supports multiple images being built in parallel (matrix strategy)
- Only builds images that have changed

#### 3. Version Extraction

The workflow parses `ARG` instructions from your Dockerfile to extract versions:

```dockerfile
ARG HUGO_VERSION=0.152.2
ARG BUN_VERSION=1.3.2
```

These are automatically extracted and used to generate version-specific tags.

#### 4. Tag Generation

Each image gets multiple tags:

- **Latest tag**: `ghcr.io/owner/image-name:latest`
- **Version-specific tag**: Based on extracted ARG versions

##### Example for `devcontainer-hugo-bun`:
```
ghcr.io/owner/devcontainer-hugo-bun:latest
ghcr.io/owner/devcontainer-hugo-bun:hugo0.152.2-bun1.3.2-alpine
```

#### 5. Build and Push

- Uses Docker Buildx for multiplatform builds
- Builds for `linux/amd64` and `linux/arm64`
- Pushes to GitHub Container Registry
- Uses GitHub Actions cache to speed up builds

### Adding Support for New Images

#### Option 1: Use Existing Patterns (Recommended)

If your image follows standard naming conventions:

1. Create your image directory: `my-new-image/`
2. Add a Dockerfile with version ARGs:
   ```dockerfile
   ARG NODE_VERSION=20.0.0
   ARG PYTHON_VERSION=3.11
   ```
3. The workflow will automatically create tags like: `node20.0.0-python3.11`

**Supported version variables** (for generic tagging):
- `NODE_VERSION`
- `PYTHON_VERSION`
- `GO_VERSION`
- `RUST_VERSION`
- `JAVA_VERSION`
- `PHP_VERSION`

#### Option 2: Add Custom Tagging Logic

For custom tag formats, edit the workflow file:

```yaml
case "$IMAGE_NAME" in
  devcontainer-hugo-bun)
    # Existing logic...
    ;;
  my-custom-image)
    # Add your custom tagging logic here
    CUSTOM_VERSION="${{ steps.versions.outputs.MY_VERSION }}"
    VERSION_TAG="custom-${CUSTOM_VERSION}"
    TAGS="$TAGS,$BASE_IMAGE:$VERSION_TAG"
    ;;
  *)
    # Generic fallback...
    ;;
esac
```

### Manual Workflow Dispatch

To manually trigger a build for a specific image:

1. Go to **Actions** → **Build and Push Multiplatform Images**
2. Click **Run workflow**
3. Enter the image directory name (e.g., `devcontainer-hugo-bun`)
4. Click **Run workflow**

### Workflow Permissions

The workflow requires the following permissions:
- `contents: read` - To checkout the repository
- `packages: write` - To push images to GHCR

These are automatically granted when using `${{ secrets.GITHUB_TOKEN }}`.

### Environment Variables

- `REGISTRY`: Container registry URL (default: `ghcr.io`)
- `IMAGE_NAMESPACE`: GitHub username/organization (automatically set to repository owner)

### Troubleshooting

#### Build fails with "No Dockerfile found"

Ensure your Dockerfile is in one of these locations:
- `image-name/.devcontainer/Dockerfile`
- `image-name/Dockerfile`

#### Tags not generated correctly

1. Check that your ARG declarations follow this format:
   ```dockerfile
   ARG VERSION_NAME=1.0.0
   ```
2. Ensure no spaces around the `=` sign
3. ARG names ending in `_VERSION` are automatically detected

#### Image not visible on GHCR

1. Check repository package settings
2. Ensure package visibility is set to public (if desired)
3. Verify the workflow had `packages: write` permission

### Best Practices

1. **Version updates**: Update versions in Dockerfile ARGs only
   - The workflow automatically extracts and uses them
   - No need to manually update the workflow file

2. **Testing changes**: Use workflow_dispatch to test builds before merging

3. **Commit messages**: Be clear when updating Dockerfiles:
   ```
   Update Hugo to 0.153.0 and Bun to 1.3.3
   ```

4. **Multiplatform testing**: The workflow verifies both amd64 and arm64 builds

### Example Workflow Run

```
1. Developer updates devcontainer-hugo-bun/.devcontainer/Dockerfile
2. Commits and pushes to main branch
3. Workflow detects change in devcontainer-hugo-bun
4. Extracts: HUGO_VERSION=0.152.2, BUN_VERSION=1.3.2
5. Builds for linux/amd64 and linux/arm64
6. Tags as:
   - ghcr.io/owner/devcontainer-hugo-bun:latest
   - ghcr.io/owner/devcontainer-hugo-bun:hugo0.152.2-bun1.3.2-alpine
7. Pushes to GHCR
8. Verifies both platform images can be pulled
```

### Advanced Configuration

#### Adding More Platforms

Edit the `platforms` in the build step:

```yaml
platforms: linux/amd64,linux/arm64,linux/arm/v7
```

#### Custom Build Arguments

Pass additional build args in the workflow:

```yaml
- name: Build and push Docker image
  uses: docker/build-push-action@v5
  with:
    build-args: |
      CUSTOM_ARG=value
      ANOTHER_ARG=value
```

#### Conditional Building

Add conditions to skip certain images:

```yaml
if: matrix.image != 'skip-this-image'
```

### Related Documentation

- [Docker Buildx Documentation](https://docs.docker.com/buildx/working-with-buildx/)
- [GitHub Container Registry](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [GitHub Actions: docker/build-push-action](https://github.com/docker/build-push-action)
