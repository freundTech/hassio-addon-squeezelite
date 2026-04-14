ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23

FROM $BUILD_FROM AS build-base
RUN --mount=type=cache,target=/etc/apk/cache \
    apk add --update-cache patch python3 py3-setuptools uv


# PortAudio
FROM build-base AS build-portaudio
# We build PortAudio from master to get native PulseAudio support and skip the
# ALSA to PulseAudio bridge for lower latency.
ARG PORTAUDIO_VERSION=8633971e14ec221804e6b67a196c879a373cc53d
ARG PORTAUDIO_HASH=d413fa9fdb4be09e62c69ad52c3300a74a0b78813d186a3f5af35471db9fb18d

RUN --mount=type=cache,target=/etc/apk/cache \
    apk add --update-cache gcc make musl-dev pulseaudio-dev && \
    curl -sSLf https://github.com/PortAudio/portaudio/archive/${PORTAUDIO_VERSION}.tar.gz -o portaudio.tar.gz && \
    echo "$PORTAUDIO_HASH portaudio.tar.gz" | sha256sum -c - && \
    tar -xzf portaudio.tar.gz && \
    cd "portaudio-$PORTAUDIO_VERSION/" && \
    ./configure --prefix=/usr --with-pulseaudio && \
    make && \
    DESTDIR=/output make install


# sendspin-cli
FROM build-base AS build-sendspin-cli
ARG SENDSPIN_CLI_VERSION=6.0.0
ARG SENDSPIN_CLI_HASH=fb0a303b641cbf138b2456f589e97b42ebc451ee338eff0f5e242ee5cd2ef5b8

ENV UV_COMPILE_BYTECODE=1
ENV UV_LINK_MODE=copy
ENV SENDSPIN_REQUIRE_C_EXT=1

COPY patches/sendspin-cli/ /patches
RUN --mount=type=cache,target=/etc/apk/cache \
    --mount=type=cache,target=/root/.cache/uv \
    apk add --no-cache gcc python3-dev musl-dev && \
    curl -Lf https://github.com/sendspin/sendspin-cli/archive/refs/tags/${SENDSPIN_CLI_VERSION}.tar.gz -o sendspin-cli.tar.gz && \
    echo "$SENDSPIN_CLI_HASH sendspin-cli.tar.gz" | sha256sum -c - && \
    tar -xzf sendspin-cli.tar.gz && \
    cd "sendspin-cli-$SENDSPIN_CLI_VERSION/" && \
    patch -p1 < /patches/0001-Add-uv.lock-for-reproducible-builds-225.patch && \
    sed -i 's/^version = "0.0.0"/version = "'"$SENDSPIN_CLI_VERSION"'"/' pyproject.toml && \
    UV_PROJECT_ENVIRONMENT=/venv uv sync --frozen --no-dev --no-managed-python --no-editable --compile-bytecode --python=/usr/bin/python


FROM $BUILD_FROM

COPY --from=build-portaudio /output/ /
COPY --from=build-sendspin-cli /venv /venv
RUN apk add --no-cache ffmpeg python3


# Copy root filesystem
COPY rootfs /

LABEL \
    org.opencontainers.image.title: "Home Assistant Add-on: Sendspin Player" \
    org.opencontainers.image.description: "Turn your Home Assistant into a Sendspin media player" \
    org.opencontainers.image.source="https://github.com/freundTech/hassio-apps-audio-receivers" \
    org.opencontainers.image.licenses="Apache-2.0" \
    org.opencontainers.authors: "Adrian Freund"
