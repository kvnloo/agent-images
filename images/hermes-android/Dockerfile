# kvnloo/hermes-android: Flutter (version from .github/workflows/pr-quality.yml), JDK 17, Android SDK.
ARG BASE=ghcr.io/kvnloo/agent-base:latest
FROM ${BASE}
USER root
ARG FLUTTER_VERSION=3.44.0
ARG CMDLINE_TOOLS=13114758
ENV JAVA_HOME=/opt/jdk ANDROID_HOME=/opt/android-sdk ANDROID_SDK_ROOT=/opt/android-sdk FLUTTER_ROOT=/opt/flutter
ENV PATH=/opt/flutter/bin:/opt/jdk/bin:/opt/android-sdk/cmdline-tools/latest/bin:/opt/android-sdk/platform-tools:${PATH}
RUN mkdir -p /opt/jdk && curl -fsSL "https://api.adoptium.net/v3/binary/latest/17/ga/linux/x64/jdk/hotspot/normal/eclipse" | tar -xz -C /opt/jdk --strip-components=1
RUN mkdir -p /opt/android-sdk/cmdline-tools && cd /tmp \
    && curl -fsSLo t.zip "https://dl.google.com/android/repository/commandlinetools-linux-${CMDLINE_TOOLS}_latest.zip" \
    && unzip -q t.zip && mv cmdline-tools /opt/android-sdk/cmdline-tools/latest && rm t.zip \
    && yes | sdkmanager --licenses >/dev/null \
    && sdkmanager "platform-tools" "platforms;android-36" "build-tools;36.0.0" >/dev/null
RUN curl -fsSL "https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz" | tar -xJ -C /opt \
    && git config --system --add safe.directory /opt/flutter \
    && flutter config --no-analytics >/dev/null && flutter precache --android >/dev/null && flutter --version
RUN chown -R hermes:hermes /opt/flutter /opt/android-sdk
