FROM python:3.12.6-alpine3.20

COPY ["rss", "rss"]
COPY ["cert", "cert"]

# Install required packages
RUN apk add --no-cache \
        boost-system \
        boost-thread \
        ca-certificates \
        dumb-init \
		icu \
		openssl \
        qt6-qtbase \
		zlib

# Compiling qBitTorrent following instructions on
# https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu#Libtorrent
RUN set -x \
    # Install build dependencies
 && apk add --no-cache -t .build-deps \
        boost-dev \
        curl \
        cmake \
        g++ \
        make \
    # Build lib rasterbar from source code (required by qBittorrent)
 && LIBTORRENT_RASTERBAR_URL="https://github.com/arvidn/libtorrent/releases/download/v2.0.10/libtorrent-rasterbar-2.0.10.tar.gz" \
 && mkdir /tmp/libtorrent-rasterbar \
 && curl -sSL $LIBTORRENT_RASTERBAR_URL | tar xzC /tmp/libtorrent-rasterbar \
 && cd /tmp/libtorrent-rasterbar/* \
 && mkdir build \
 && cd build \
 && cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_STANDARD=20 -UCMAKE_INSTALL_LIBDIR -DCMAKE_INSTALL_LIBDIR=lib .. \
 && make install \
    # Clean-up
 && cd / \
 && apk del --purge .build-deps \
 && rm -rf /tmp/*

RUN set -x \
    # Install build dependencies
 && apk add --no-cache -t .build-deps \
        autoconf \
		automake \
		boost-dev \
        build-base \
		cmake \
		curl \
		g++ \
        git \
		libtool \
		linux-headers \
		make \
		perl \
		pkgconf \
		python3 \
		python3-dev \
        qt6-qttools-dev \
		re2c \
		samurai \
		tar \
    # Build qBittorrent from source code
 && git clone https://github.com/qbittorrent/qBittorrent.git /tmp/qbittorrent \
 && cd /tmp/qbittorrent \
 && git checkout release-5.0.0 \
    # Compile
 && cmake -Wno-dev -G Ninja -B build \
    -D CMAKE_BUILD_TYPE="Release" \
    -D CMAKE_CXX_STANDARD=20 \
    -D CMAKE_INSTALL_LIBDIR="lib" \
    -D CMAKE_INSTALL_PREFIX="/usr/local" \
	-D GUI="OFF" \
 && cmake --build build \
 && cmake --install build \
    # Clean-up
 && cd / \
 && apk del --purge .build-deps \
 && rm -rf /tmp/* \
    # Add non-root user
 && adduser -S -D -u 520 -g 520 -s /sbin/nologin qbittorrent \
    # Create symbolic links to simplify mounting
 && mkdir -p /home/qbittorrent/.config/qBittorrent \
 && mkdir -p /home/qbittorrent/.local/share/data/qBittorrent \
 && mkdir /downloads \
 && chmod go+rw -R /home/qbittorrent /downloads \
 && ln -s /home/qbittorrent/.config/qBittorrent /config \
 && ln -s /home/qbittorrent/.local/share/data/qBittorrent /torrents \
 && mkdir /scan && mkdir /scan/1 && mkdir /scan/2 && mkdir /scan/3 \
    # Check it works
 && su qbittorrent -s /bin/sh -c 'qbittorrent-nox -v'

# Default configuration file.
COPY 4.6.0/qBittorrent.conf /config/qBittorrent.conf
COPY entrypoint.sh /

ENV HOME=/home/qbittorrent

USER qbittorrent

EXPOSE 8080 6881

VOLUME /data

ENTRYPOINT ["dumb-init", "/entrypoint.sh"]
CMD ["qbittorrent-nox"]
