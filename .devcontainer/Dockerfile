# Update the NODE_VERSION arg in docker-compose.yml to pick a Node version: 18, 16, 14
ARG NODE_VERSION=22
FROM mcr.microsoft.com/devcontainers/javascript-node:${NODE_VERSION}

# VARIANT can be either 'hugo' for the standard version or 'hugo_extended' for the extended version.
ARG VARIANT=hugo_extended
# VERSION can be either 'latest' or a specific version number
ARG VERSION=latest

# Install Go and build tools needed to build Hugo from source
# For extended variant, also install CGO dependencies (libc6-dev, gcc, g++, libstdc++6)
RUN apt-get update && apt-get install -y ca-certificates openssl git curl build-essential gcc g++ libc6-dev && \
    rm -rf /var/lib/apt/lists/*

# Install Go (required to build Hugo from source)
RUN ARCH=$(dpkg --print-architecture) && \
    case ${ARCH} in \
    amd64) GOARCH=amd64 ;; \
    arm64) GOARCH=arm64 ;; \
    *) echo "Unsupported architecture: ${ARCH}" && exit 1 ;; \
    esac && \
    curl -fsSL https://go.dev/dl/go1.21.5.linux-${GOARCH}.tar.gz -o go.tar.gz && \
    tar -C /usr/local -xzf go.tar.gz && \
    rm go.tar.gz

ENV PATH="/usr/local/go/bin:${PATH}"

# Build Hugo from source to ensure GLIBC compatibility
RUN case ${VERSION} in \
    latest) \
    export VERSION=$(curl -s https://api.github.com/repos/gohugoio/hugo/releases/latest | grep "tag_name" | awk '{print substr($2, 3, length($2)-4)}') ;;\
    esac && \
    echo "Building Hugo version ${VERSION}" && \
    git clone --depth 1 --branch v${VERSION} https://github.com/gohugoio/hugo.git /tmp/hugo && \
    cd /tmp/hugo && \
    if [ "${VARIANT}" = "hugo_extended" ]; then \
        CGO_ENABLED=1 go build -tags extended -o /usr/bin/hugo; \
    else \
        go build -o /usr/bin/hugo; \
    fi && \
    rm -rf /tmp/hugo && \
    hugo version

# Hugo dev server port
EXPOSE 1313

# [Optional] Uncomment this section to install additional OS packages you may want.
#
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment if you want to install more global node packages
# RUN sudo -u node npm install -g <your-package-list-here>
