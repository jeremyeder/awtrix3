FROM mcr.microsoft.com/devcontainers/cpp:1-debian-12

ARG REINSTALL_CMAKE_VERSION_FROM_SOURCE="none"

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        python3-venv \
        python3-pip \
        python3-full \
        git \
        curl \
        jq \
        clangd \
        udev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install PlatformIO CLI
RUN python3 -m pip install --break-system-packages --upgrade pip \
    && python3 -m pip install --break-system-packages platformio

# Pre-install ESP32 platform and toolchain so builds work immediately
RUN platformio platform install "espressif32@6.3.0" --with-package toolchain-xtensa-esp32 \
    && platformio platform install "espressif32" --with-package toolchain-xtensa-esp32
