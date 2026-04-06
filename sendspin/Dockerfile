ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23

FROM $BUILD_FROM AS build-base
RUN apk add --no-cache python3 py3-setuptools uv

FROM build-base AS build-native
RUN apk add --no-cache gcc python3-dev musl-dev

# PortAudio
FROM build-native AS build-portaudio
# We build PortAudio from master to get native PulseAudio support and skip the
# ALSA to PulseAudio bridge for lower latency.
ARG PORTAUDIO_VERSION=8633971e14ec221804e6b67a196c879a373cc53d
ARG PORTAUDIO_HASH=d413fa9fdb4be09e62c69ad52c3300a74a0b78813d186a3f5af35471db9fb18d

RUN apk add --no-cache make pulseaudio-dev && \
    curl -sSLf https://github.com/PortAudio/portaudio/archive/${PORTAUDIO_VERSION}.tar.gz -o portaudio.tar.gz && \
    echo "$PORTAUDIO_HASH portaudio.tar.gz" | sha256sum -c - && \
    tar -xzf portaudio.tar.gz && \
    cd "portaudio-$PORTAUDIO_VERSION/" && \
    ./configure --prefix=/usr --with-pulseaudio && \
    make && \
    DESTDIR=/output make install

# PyAV
FROM build-native AS build-pyav
# PyAV doesn't have a musl wheel and is not packaged by Alpine.
ARG PYAV_VERSION=16.1.0
ARG PYAV_HASH=9a73e1955c1d0e6d1228f3f566326f58be4303614b04e4549c2c1a93a0c9ee58

RUN apk add --no-cache cython ffmpeg-dev && \
    curl -sSLf https://github.com/PyAV-Org/PyAV/archive/refs/tags/v${PYAV_VERSION}.tar.gz -o PyAV.tar.gz && \
    echo "$PYAV_HASH PyAV.tar.gz" | sha256sum -c - && \
    tar -xzf PyAV.tar.gz && \
    cd "PyAV-$PYAV_VERSION/" && \
    uv build --offline --no-build-isolation --wheel --out-dir=/wheels


# aiosendspin
FROM build-base AS build-aiosendspin
ARG AIOSENDSPIN_VERSION=4.4.0
ARG AIOSENDSPIN_HASH=30c3f8fa2bf5f31056b13338b180bbd9bd17c17b5f729897ce310111d9065b1e


COPY patches/aiosendspin/ /patches
RUN apk add --no-cache patch && \
    curl -sSLf https://github.com/sendspin/aiosendspin/archive/refs/tags/${AIOSENDSPIN_VERSION}.tar.gz -o aiosendspin.tar.gz && \
    echo "$AIOSENDSPIN_HASH aiosendspin.tar.gz" | sha256sum -c - && \
    tar -xzf aiosendspin.tar.gz && \
    cd "aiosendspin-$AIOSENDSPIN_VERSION/" && \
    patch -p1 < /patches/0001-Fix-wrong-connection-check-in-send_message.patch && \
    sed -i 's/^version = "0.0.0"/version = "'"$AIOSENDSPIN_VERSION"'"/' pyproject.toml && \
    uv build --offline --no-build-isolation --wheel --out-dir=/wheels


# aiosendspin-mpris
FROM build-base AS build-aiosendspin-mpris
ARG AIOSENDSPIN_MPRIS_VERSION=2.1.1
ARG AIOSENDSPIN_MPRIS_HASH=5b02ce42019955f1f1e6ae1d1fb5ad527bf4f1638bc05e601e738d58e3a97e6b

RUN \
    curl -sSLf https://github.com/abmantis/aiosendspin-mpris/archive/refs/tags/v${AIOSENDSPIN_MPRIS_VERSION}.tar.gz -o aiosendspin-mpris.tar.gz && \
    echo "$AIOSENDSPIN_MPRIS_HASH aiosendspin-mpris.tar.gz" | sha256sum -c - && \
    tar -xzf aiosendspin-mpris.tar.gz && \
    cd "aiosendspin-mpris-$AIOSENDSPIN_MPRIS_VERSION/" && \
    uv build --offline --no-build-isolation --wheel --out-dir=/wheels


# sendspin-cli
FROM build-native AS build-sendspin-cli
ARG SENDSPIN_CLI_VERSION=5.9.0
ARG SENDSPIN_CLI_HASH=0fec3c1de890374a1fa69f9164df507518da68d30652dfb271ab1d090b81c53b

RUN curl -Lf https://github.com/sendspin/sendspin-cli/archive/refs/tags/${SENDSPIN_CLI_VERSION}.tar.gz -o sendspin-cli.tar.gz && \
    echo "$SENDSPIN_CLI_HASH sendspin-cli.tar.gz" | sha256sum -c - && \
    tar -xzf sendspin-cli.tar.gz && \
    cd "sendspin-cli-$SENDSPIN_CLI_VERSION/" && \
    sed -i 's/^version = "0.0.0"/version = "'"$SENDSPIN_CLI_VERSION"'"/' pyproject.toml && \
    SENDSPIN_REQUIRE_C_EXT=1 uv build --offline --no-build-isolation --wheel --out-dir=/wheels

FROM $BUILD_FROM

COPY --from=build-portaudio /output/ /

RUN --mount=type=bind,from=build-pyav,source=/wheels,target=/wheels/pyav \
    --mount=type=bind,from=build-aiosendspin,source=/wheels,target=/wheels/aiosendspin\
    --mount=type=bind,from=build-aiosendspin-mpris,source=/wheels,target=/wheels/aiosendspin-mpris\
    --mount=type=bind,from=build-sendspin-cli,source=/wheels,target=/wheels/sendspin-cli\
    # zeroconf doesn't have a musl wheel, so we get it from Alpine
    apk add --no-cache ffmpeg python3 py3-zeroconf uv  && \
    uv pip install --system --break-system-packages --no-cache /wheels/pyav/*.whl /wheels/aiosendspin/*.whl /wheels/aiosendspin-mpris/*.whl /wheels/sendspin-cli/*.whl && \
    apk del uv


# Copy root filesystem
COPY rootfs /

LABEL \
    org.opencontainers.image.title: "Home Assistant Add-on: Sendspin Player" \
    org.opencontainers.image.description: "Turn your Home Assistant into a Sendspin media player" \
    org.opencontainers.image.source="https://github.com/freundTech/hassio-apps-audio-receivers" \
    org.opencontainers.image.licenses="Apache-2.0" \
    org.opencontainers.authors: "Adrian Freund"
