# kvnloo/glide: uv (in base) fetches Python 3.14 on demand; pre-warm it.
ARG BASE=ghcr.io/kvnloo/agent-base:latest
FROM ${BASE}
USER root
ENV UV_PYTHON_INSTALL_DIR=/opt/uv-python
RUN uv python install 3.14 && chmod -R a+rX /opt/uv-python
