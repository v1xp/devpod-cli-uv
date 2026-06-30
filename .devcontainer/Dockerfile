FROM ghcr.io/astral-sh/uv:latest AS uv

FROM mcr.microsoft.com/devcontainers/base:ubuntu AS nvim-builder

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
  && NVIM_TAG=$(curl -fsSL https://api.github.com/repos/neovim/neovim/releases/latest | grep '"tag_name"' | cut -d '"' -f 4) \
  && echo "Building Neovim $NVIM_TAG" \
  && git clone --depth 1 --branch "$NVIM_TAG" https://github.com/neovim/neovim /tmp/neovim \
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
  stow \
  ca-certificates \
  curl \
  gnupg \
  lsb-release \
  && install -m 0755 -d /etc/apt/keyrings \
  && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg \
  && chmod a+r /etc/apt/keyrings/docker.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
  && apt-get update \
  && apt-get -y install --no-install-recommends docker-ce-cli docker-compose-plugin \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://opencode.ai/install | bash \
  && cp /root/.opencode/bin/opencode /usr/local/bin/opencode

ARG LAZYGIT_VERSION=0.44.1
ARG TARGETARCH
RUN LAZYGIT_ARCH="${TARGETARCH}" \
  && if [ "$LAZYGIT_ARCH" = "amd64" ]; then LAZYGIT_ARCH="x86_64"; fi \
  && curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/download/v${LAZYGIT_VERSION}/lazygit_${LAZYGIT_VERSION}_Linux_${LAZYGIT_ARCH}.tar.gz" \
  && tar xf lazygit.tar.gz lazygit \
  && install lazygit /usr/local/bin/ \
  && rm -f lazygit lazygit.tar.gz

# Install act (GitHub Actions local runner)
ARG ACT_VERSION=0.2.79
RUN ACT_ARCH="${TARGETARCH}" \
  && if [ "$ACT_ARCH" = "amd64" ]; then ACT_ARCH="x86_64"; fi \
  && curl -Lo act.tar.gz "https://github.com/nektos/act/releases/download/v${ACT_VERSION}/act_Linux_${ACT_ARCH}.tar.gz" \
  && tar xf act.tar.gz act \
  && install act /usr/local/bin/ \
  && rm -f act act.tar.gz

# Copy built neovim from builder
COPY --from=nvim-builder /usr/local/bin/nvim /usr/local/bin/nvim
COPY --from=nvim-builder /usr/local/lib/nvim /usr/local/lib/nvim
COPY --from=nvim-builder /usr/local/share/nvim /usr/local/share/nvim

# Install uv (Astral's ultra-fast Python package manager)
COPY --from=uv /uv /uvx /usr/local/bin/

# Ensure the vscode/devcontainer user has ownership and fix /root traversal
USER root
RUN groupadd -f docker && usermod -aG docker vscode \
  && mkdir -p /home/vscode && chown vscode:vscode /home/vscode \
  && chmod 755 /root && git init /root

USER vscode
RUN git config --global user.name "vscode" && git config --global user.email "vscode@devcontainer.local"
ENV PATH="/home/vscode/.local/bin:${PATH}"
