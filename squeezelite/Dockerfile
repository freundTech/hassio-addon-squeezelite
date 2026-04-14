ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23
FROM $BUILD_FROM AS build

# Version 2.0.1556
ARG SQUEEZELITE_COMMIT=e70c19f79f76209b05806e022a9959c96d32697c

RUN apk add build-base pulseaudio-dev flac-dev libvorbis-dev faad2-dev libmad-dev mpg123-dev
RUN \
    curl -sSLf https://github.com/ralph-irving/squeezelite/archive/${SQUEEZELITE_COMMIT}.tar.gz -o squeezelite.tar.gz&& \
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
