# kvnloo/dash: Bun (version from .github/workflows/ci.yml) on the base's Node.
ARG BASE=ghcr.io/kvnloo/agent-base:latest
FROM ${BASE}
USER root
ARG BUN_VERSION=1.4.3
RUN curl -fsSL https://bun.sh/install | BUN_INSTALL=/usr/local bash -s "bun-v${BUN_VERSION}" && bun --version
