ARG BUN_VERSION=1.3.9

FROM oven/bun:${BUN_VERSION}-alpine

ARG HUGO_VERSION=0.155.3
ARG RALPHEX_VERSION=0.11.0

# Platform architecture (automatically set by Docker based on build platform)
ARG TARGETARCH

# Install dependencies:
# - ca-certificates: Required for HTTPS connections
# - gcompat: glibc compatibility layer required for Hugo Extended binary (Alpine uses musl)
# - go: Required for Hugo Modules to download and manage dependencies (e.g., PaperMod theme)
# - wget: Required to download Hugo and Ralphex binaries
RUN apk add --no-cache ca-certificates gcompat go wget

# Install Ralphex by downloading pre-built binary with checksum verification
RUN set -eux; \
    wget -O ralphex.tar.gz "https://github.com/umputun/ralphex/releases/download/v${RALPHEX_VERSION}/ralphex_${RALPHEX_VERSION}_linux_${TARGETARCH}.tar.gz"; \
    wget -O ralphex_checksums.txt "https://github.com/umputun/ralphex/releases/download/v${RALPHEX_VERSION}/ralphex_${RALPHEX_VERSION}_checksums.txt"; \
    EXPECTED_CHECKSUM=$(grep "ralphex_${RALPHEX_VERSION}_linux_${TARGETARCH}.tar.gz" ralphex_checksums.txt | cut -d' ' -f1); \
    if [ -z "$EXPECTED_CHECKSUM" ]; then \
        echo "Error: Could not find checksum for ralphex_${RALPHEX_VERSION}_linux_${TARGETARCH}.tar.gz"; \
        exit 1; \
    fi; \
    ACTUAL_CHECKSUM=$(sha256sum ralphex.tar.gz | cut -d' ' -f1); \
    if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]; then \
        echo "Checksum verification failed for Ralphex!"; \
        echo "Expected: $EXPECTED_CHECKSUM"; \
        echo "Actual:   $ACTUAL_CHECKSUM"; \
        exit 1; \
    fi; \
    echo "Ralphex checksum verified: $ACTUAL_CHECKSUM"; \
    tar -xzf ralphex.tar.gz -C /usr/local/bin/ ralphex; \
    rm ralphex.tar.gz ralphex_checksums.txt; \
    ralphex --version

# Install Hugo Extended by downloading pre-built binary with checksum verification
RUN set -eux; \
    wget -O hugo.tar.gz "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-${TARGETARCH}.tar.gz"; \
    wget -O hugo_checksums.txt "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_checksums.txt"; \
    EXPECTED_CHECKSUM=$(grep "hugo_extended_${HUGO_VERSION}_linux-${TARGETARCH}.tar.gz" hugo_checksums.txt | cut -d' ' -f1); \
    if [ -z "$EXPECTED_CHECKSUM" ]; then \
        echo "Error: Could not find checksum for hugo_extended_${HUGO_VERSION}_linux-${TARGETARCH}.tar.gz"; \
        exit 1; \
    fi; \
    ACTUAL_CHECKSUM=$(sha256sum hugo.tar.gz | cut -d' ' -f1); \
    if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]; then \
        echo "Checksum verification failed for Hugo!"; \
        echo "Expected: $EXPECTED_CHECKSUM"; \
        echo "Actual:   $ACTUAL_CHECKSUM"; \
        exit 1; \
    fi; \
    echo "Hugo checksum verified: $ACTUAL_CHECKSUM"; \
    tar -xzf hugo.tar.gz -C /usr/local/bin/ hugo; \
    rm hugo.tar.gz hugo_checksums.txt; \
    hugo version

# OCI labels for container metadata and GitHub Package integration
LABEL org.opencontainers.image.source="https://github.com/gatezh/devcontainer-images"
LABEL org.opencontainers.image.description="Ralphex-fe: Frontend development image with Bun, Hugo Extended, and Ralphex"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.title="ralphex-fe"
LABEL org.opencontainers.image.url="https://github.com/gatezh/devcontainer-images"
