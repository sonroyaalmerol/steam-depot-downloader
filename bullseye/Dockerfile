FROM --platform=arm64 python:slim-bullseye as box64_m1

ENV DEBIAN_FRONTEND noninteractive

# Set SHELL option explicitly
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
        git \
		ca-certificates \
        build-essential \
        cmake \
	&& git clone https://github.com/ptitSeb/box64.git \
    && mkdir /box64/build \
	&& cd /box64/build \
	&& cmake .. -D M1=1 -D CMAKE_BUILD_TYPE=RelWithDebInfo \
	&& make -j$(nproc) \
	&& make install DESTDIR=/tmp/install

############################################################

FROM --platform=arm64 python:slim-bullseye as box64_rpi5

ENV DEBIAN_FRONTEND noninteractive

# Set SHELL option explicitly
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
        git \
		ca-certificates \
        build-essential \
        cmake \
	&& git clone https://github.com/ptitSeb/box64.git \
    && mkdir /box64/build \
	&& cd /box64/build \
	&& cmake .. -D RPI5ARM64=1 -D CMAKE_BUILD_TYPE=RelWithDebInfo \
	&& make -j$(nproc) \
	&& make install DESTDIR=/tmp/install

############################################################

FROM --platform=arm64 python:slim-bullseye as box64_adlink

ENV DEBIAN_FRONTEND noninteractive

# Set SHELL option explicitly
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		git \
		ca-certificates \
		build-essential \
		cmake \
	&& git clone https://github.com/ptitSeb/box64.git \
	&& mkdir /box64/build \
	&& cd /box64/build \
	&& cmake .. -D ADLINK=1 -D CMAKE_BUILD_TYPE=RelWithDebInfo \
	&& make -j$(nproc) \
	&& make install DESTDIR=/tmp/install

############################################################
# Dockerfile that contains Depot Downloader and Box86/64
############################################################

FROM debian:bullseye-slim as build_stage_amd64

ARG TARGETARCH
ARG DEPOT_DOWNLOADER_VERSION="2.6.0"

LABEL maintainer="github@snry.me"

ENV DEBIAN_FRONTEND noninteractive

# Set SHELL option explicitly
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN set -x \
	# Install, update & upgrade packages
	&& apt-get update \
 	&& apt-get install -y --no-install-recommends --no-install-suggests \
		openssl \
		ca-certificates \
		nano \
		curl \
		locales \
  		wget \
		gnupg \
        libicu67 \
        unzip \
	&& sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
	&& dpkg-reconfigure --frontend=noninteractive locales \
	&& rm -rf /var/lib/apt/lists/* \
    && case "${TARGETARCH}" in \
    "amd64") DEPOT_DOWNLOADER_FILENAME=DepotDownloader-linux-x64.zip ;; \
    "arm64") DEPOT_DOWNLOADER_FILENAME=DepotDownloader-linux-arm64.zip ;; \
    esac \
    && wget --progress=dot:giga "https://github.com/SteamRE/DepotDownloader/releases/download/DepotDownloader_${DEPOT_DOWNLOADER_VERSION}/${DEPOT_DOWNLOADER_FILENAME}" -O DepotDownloader.zip \
    && unzip DepotDownloader.zip \
    && rm -rf DepotDownloader.xml \
    && chmod +x DepotDownloader \
    && mv DepotDownloader /usr/local/bin/DepotDownloader

FROM --platform=arm64 debian:bullseye-slim as build_stage_arm64

COPY --from=box64_m1 /tmp/install/usr/local/bin/box64 /usr/local/bin/box64-m1
COPY --from=box64_rpi5 /tmp/install/usr/local/bin/box64 /usr/local/bin/box64-rpi5
COPY --from=box64_adlink /tmp/install/usr/local/bin/box64 /usr/local/bin/box64-adlink

# hadolint ignore=DL3008
RUN set -x \
	# Install, update & upgrade packages
	&& dpkg --add-architecture armhf \
	&& dpkg --add-architecture i386 \
	&& wget --progress=dot:giga https://ryanfortner.github.io/box64-debs/box64.list -O /etc/apt/sources.list.d/box64.list \
	&& (wget -qO- https://ryanfortner.github.io/box64-debs/KEY.gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/box64-debs-archive-keyring.gpg) \
	&& apt-get update \
 	&& apt-get install -y --no-install-recommends --no-install-suggests \
		libc6:i386 \
		libc6:armhf \
		libcurl4 \
		libcurl4:i386 \
		libnuma1 \
		libnuma1:i386 \
		libglib2.0-0 \
		libglib2.0-0:i386 \
		box64 \
	&& sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
	&& dpkg-reconfigure --frontend=noninteractive locales \
	&& rm -rf /var/lib/apt/lists/* \
	&& mv /usr/local/bin/box64 /usr/local/bin/box64-generic

COPY box64.sh /usr/local/bin/box64
RUN chmod +x /usr/local/bin/box64

ARG TARGETARCH

FROM build_stage_${TARGETARCH} AS bullseye