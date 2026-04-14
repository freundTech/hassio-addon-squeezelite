ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23
FROM $BUILD_FROM AS build

# Version 2.0.1561
ARG SQUEEZELITE_COMMIT=e977d1045f5f4c1a51ba0d66387f26fd19a2f42a
ARG SQUEEZELITE_HASH=32f0b25888e400dcf571f9159079db1275a674d43e8567217a5d673fd38f4cca

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
