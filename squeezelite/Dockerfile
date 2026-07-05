ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23
FROM $BUILD_FROM AS build

# Version 2.0.1577
ARG SQUEEZELITE_COMMIT=d0d17404467bc18326d9de94eaf3949cf8fb8f59
ARG SQUEEZELITE_HASH=0dcaaae7916edfa794e28d0afd64fac6cd93cb69996539e35c1d85f25b7cfb0d

RUN apk add build-base pulseaudio-dev flac-dev libvorbis-dev faad2-dev libmad-dev mpg123-dev
RUN \
    curl -sSLf https://github.com/ralph-irving/squeezelite/archive/${SQUEEZELITE_COMMIT}.tar.gz -o squeezelite.tar.gz && \
    echo "$SQUEEZELITE_HASH squeezelite.tar.gz" | sha256sum -c - && \
    tar -xzvf squeezelite.tar.gz && \
    cd squeezelite-${SQUEEZELITE_COMMIT}/ && \
    make -f Makefile.pulse

FROM $BUILD_FROM

COPY --from=build /squeezelite-*/squeezelite-pulse /usr/bin/squeezelite

RUN apk add pulseaudio libflac libvorbis faad2 libmad mpg123

# Copy root filesystem
COPY rootfs /

LABEL \
    org.opencontainers.image.title="Home Assistant Add-on: Squeezelite Player" \
    org.opencontainers.image.description="Turn your Home Assistant into a Squeezelite media player" \
    org.opencontainers.image.source="https://github.com/freundTech/hassio-apps-audio-receivers" \
    org.opencontainers.image.licenses="GPL-3.0-or-later" \
    org.opencontainers.authors="Adrian Freund"
