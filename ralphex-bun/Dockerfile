ARG BUN_VERSION=1.3.8
ARG HUGO_VERSION=0.155.2

FROM ghcr.io/umputun/ralphex:latest

ARG BUN_VERSION
ARG HUGO_VERSION

# Platform architecture (automatically set by Docker based on build platform)
ARG TARGETARCH

# Install dependencies:
# - gcompat: glibc compatibility layer required for Hugo Extended binary (Alpine uses musl)
# - go: Required for Hugo Modules to download and manage dependencies (e.g., PaperMod theme)
# - unzip: Required by Bun install script on Linux
# - wget: Required to download Hugo Extended binary
RUN apk add --no-cache gcompat go unzip wget

# Install Bun using official install script
# BUN_INSTALL sets the installation directory
ENV BUN_INSTALL=/usr/local/bun
ENV PATH="$BUN_INSTALL/bin:$PATH"

RUN curl -fsSL https://bun.sh/install | bash -s "bun-v${BUN_VERSION}" && \
    bun --version

# Install Hugo Extended by downloading pre-built binary
RUN set -eux; \
    wget -O hugo.tar.gz "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-${TARGETARCH}.tar.gz"; \
    tar -xzf hugo.tar.gz -C /usr/local/bin/ hugo; \
    rm hugo.tar.gz; \
    hugo version

# OCI labels for container metadata and GitHub Package integration
LABEL org.opencontainers.image.source="https://github.com/gatezh/devcontainer-images"
LABEL org.opencontainers.image.description="Ralphex-based image with Bun and Hugo Extended - multiplatform image for modern development"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.title="ralphex-bun"
LABEL org.opencontainers.image.url="https://github.com/gatezh/devcontainer-images"
