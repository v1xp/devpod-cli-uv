FROM mcr.microsoft.com/devcontainers/base:ubuntu AS nvim-builder

ARG NVIM_VERSION=v0.12.3

USER root

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
  && apt-get -y install --no-install-recommends \
  git \
  cmake \
  ninja-build \
  gettext \
  libtool \
  libtool-bin \
  autoconf \
  automake \
  g++ \
  pkg-config \
  unzip \
  curl \
  build-essential \
  && git clone --depth 1 --branch ${NVIM_VERSION} https://github.com/neovim/neovim /tmp/neovim \
  && cd /tmp/neovim \
  && make CMAKE_BUILD_TYPE=Release CMAKE_INSTALL_PREFIX=/usr/local \
  && make install \
  && rm -rf /tmp/neovim

# --- Final stage ---
FROM mcr.microsoft.com/devcontainers/base:ubuntu

USER root

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
  && apt-get -y install --no-install-recommends \
  tmux \
  gpg \
  gnupg2 \
  pass \
  curl \
  git \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://opencode.ai/install | bash

# Copy built neovim from builder
COPY --from=nvim-builder /usr/local/bin/nvim /usr/local/bin/nvim
COPY --from=nvim-builder /usr/local/lib/nvim /usr/local/lib/nvim
COPY --from=nvim-builder /usr/local/share/nvim /usr/local/share/nvim

# Install uv (Astral's ultra-fast Python package manager)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/

# Ensure the vscode/devcontainer user has ownership
USER vscode
ENV PATH="/home/vscode/.local/bin:${PATH}"
