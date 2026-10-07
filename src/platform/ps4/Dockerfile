FROM ubuntu:20.04 AS tools

ARG DEBIAN_FRONTEND=noninteractive
ARG SHADERC_COMMIT=d5f08ae5c5a9a45165578445cbd0f9adf0223448
ARG PSBC_COMMIT=e21e69960f387121428521988237744f6111f13f
ARG OPENGNM_COMMIT=cb381c485ca7e2c1d1c34409ee4720ae2e65806c
ARG VULKAN_HEADERS_COMMIT=8c2683390a4b10c0483e3b531555dfb841e2cd84

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        clang \
        cmake \
        git \
        make \
        ninja-build \
        pkg-config \
        python3 \
        python3-pip \
        wget \
        xz-utils && \
    rm -rf /var/lib/apt/lists/*

# Build shaderc / glslc
RUN git clone https://github.com/google/shaderc.git && \
    cd shaderc && \
    git checkout "${SHADERC_COMMIT}" && \
    ./utils/git-sync-deps && \
    cmake -S . -B build -G Ninja \
        -DSHADERC_SKIP_TESTS=ON \
        -DSHADERC_SKIP_EXAMPLES=OFF \
        -DSHADERC_SKIP_COPYRIGHT_CHECK=ON && \
    cmake --build build -j"$(nproc)" && \
    install -m 0755 build/glslc/glslc /usr/local/bin/glslc

COPY src/platform/ps4/psbc-opengnm.patch /tmp/psbc-opengnm.patch

RUN git clone https://github.com/bizkut/psbc.git && \
    cd psbc && \
    git checkout "${PSBC_COMMIT}" && \
    git apply /tmp/psbc-opengnm.patch && \
    cd .. && \
    git clone https://github.com/PS4-OpenGNM/opengnm.git OpenGNM && \
    cd OpenGNM && \
    git checkout "${OPENGNM_COMMIT}" && \
    cd .. && \
    git clone https://github.com/KhronosGroup/Vulkan-Headers.git Vulkan-Headers && \
    cd Vulkan-Headers && \
    git checkout "${VULKAN_HEADERS_COMMIT}" && \
    cd .. && \
    mkdir -p opengnm-psbc/include && \
    make -C psbc -j"$(nproc)" GNM_INCLUDE=../OpenGNM/include && \
    install -m 0755 psbc/psbc /usr/local/bin/psbc


FROM openorbisofficial/toolchain@sha256:f752d09f9d3227c729ee069f29a05450bd8fa4ef5e785c0929ab8479f7dc4e64

USER root

ARG DEBIAN_FRONTEND=noninteractive
ARG OPENGNM_COMMIT=cb381c485ca7e2c1d1c34409ee4720ae2e65806c

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        cmake \
        git \
        make \
        ninja-build \
        pkg-config \
        python3 && \
    rm -rf /var/lib/apt/lists/*

COPY --from=tools /usr/local/bin/glslc /usr/local/bin/glslc
COPY --from=tools /usr/local/bin/psbc /usr/local/bin/psbc

COPY src/platform/ps4/opengnm-orbis-stubs.patch /tmp/opengnm-orbis-stubs.patch

RUN git clone https://github.com/PS4-OpenGNM/opengnm.git /opt/opengnm && \
    cd /opt/opengnm && \
    git checkout "${OPENGNM_COMMIT}" && \
    git apply /tmp/opengnm-orbis-stubs.patch

ENV OPENORBIS=/opt/openorbis
ENV OO_PS4_TOOLCHAIN=/opt/openorbis

WORKDIR /home/mgba/src
