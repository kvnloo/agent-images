# kvnloo/hermex (iOS): Linux-side checks only (fork-logic harness, pbxproj via ruby xcodeproj,
# string catalog). Compiling Swift for iOS still needs macOS CI.
ARG BASE=ghcr.io/kvnloo/agent-base:latest
FROM ${BASE}
USER root
RUN apt-get update && apt-get install -y --no-install-recommends ruby-full && rm -rf /var/lib/apt/lists/* \
    && gem install --no-document xcodeproj
