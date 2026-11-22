# Hermes + Claude Code + the DirectSDK provider. Every repo image builds on this.
# Upstream nousresearch/hermes-agent does the heavy lifting; we only add what lanes need.
ARG HERMES_TAG=stable
FROM nousresearch/hermes-agent:${HERMES_TAG}
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
      gh jq ripgrep unzip xz-utils zip curl ca-certificates build-essential tmux \
    && rm -rf /var/lib/apt/lists/*
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/
RUN npm install -g @anthropic-ai/claude-code && claude --version
# DirectSDK provider plugin, tracked from upstream main on every (weekly) rebuild.
RUN git clone -q --depth 1 --filter=blob:none --sparse https://github.com/NousResearch/hermes-official-plugins /tmp/hop \
    && git -C /tmp/hop sparse-checkout set claude-subscription-directsdk \
    && mkdir -p /opt/plugins && mv /tmp/hop/claude-subscription-directsdk /opt/plugins/claude-subscription-directsdk-experimental \
    && rm -rf /tmp/hop
COPY agent-setup lane /usr/local/bin/
RUN chmod +x /usr/local/bin/agent-setup /usr/local/bin/lane
LABEL org.opencontainers.image.source=https://github.com/kvnloo/agent-images
