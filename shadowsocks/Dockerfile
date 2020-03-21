FROM alpine:3.11

MAINTAINER qiudog825@gmail.com

ARG TZ='Asia/Chongqing'
ARG SS_LIBEV_VERSION=v3.3.3
ARG KCP_VERSION=20200321
ARG V2RAY_PLUGIN_VERSION=v1.3.0
ARG SS_DOWNLOAD_URL=https://github.com/shadowsocks/shadowsocks-libev.git 
ARG KCP_DOWNLOAD_URL=https://github.com/xtaci/kcptun/releases/download/v${KCP_VERSION}/kcptun-linux-arm64-${KCP_VERSION}.tar.gz
ARG PLUGIN_OBFS_DOWNLOAD_URL=https://github.com/shadowsocks/simple-obfs.git
ARG PLUGIN_V2RAY_DOWNLOAD_URL=https://github.com/shadowsocks/v2ray-plugin/releases/download/${V2RAY_PLUGIN_VERSION}/v2ray-plugin-linux-arm64-${V2RAY_PLUGIN_VERSION}.tar.gz
ARG LINUX_HEADERS_DOWNLOAD_URL=http://dl-cdn.alpinelinux.org/alpine/v3.11/main/aarch64/linux-headers-4.19.36-r0.apk

RUN set -x \
    && apk upgrade \
    && apk add bash tzdata rng-tools runit \
    && apk add --virtual .build-deps \
        autoconf \
        automake \
        build-base \
        curl \
        c-ares-dev \
        libev-dev \
        libtool \
        libcap \
        libsodium-dev \
        mbedtls-dev \
        pcre-dev \
        tar \
        git \
    && curl -sSL ${LINUX_HEADERS_DOWNLOAD_URL} > /linux-headers-4.19.36-r0.apk \
    && apk add --virtual .build-deps-kernel /linux-headers-4.19.36-r0.apk \
    && git clone ${SS_DOWNLOAD_URL} \
    && (cd shadowsocks-libev \
    && git checkout tags/${SS_LIBEV_VERSION} -b ${SS_LIBEV_VERSION} \
    && git submodule update --init --recursive \
    && ./autogen.sh \
    && ./configure --prefix=/usr --disable-documentation \
    && make install) \
    && git clone ${PLUGIN_OBFS_DOWNLOAD_URL} \
    && (cd simple-obfs \
    && git submodule update --init --recursive \
    && ./autogen.sh \
    && ./configure --disable-documentation \
    && make install) \
    && curl -o v2ray_plugin.tar.gz -sSL ${PLUGIN_V2RAY_DOWNLOAD_URL} \
    && tar -zxf v2ray_plugin.tar.gz \
    && mv v2ray-plugin_linux_arm64 /usr/bin/v2ray-plugin \
    && curl -o kcptun-linux-arm64-${KCP_VERSION}.tar.gz -sSL ${KCP_DOWNLOAD_URL} \
    && tar -zxf kcptun-linux-arm64-${KCP_VERSION}.tar.gz \
    && mv server_linux_arm64 /usr/bin/kcpserver \
    && mv client_linux_arm64 /usr/bin/kcpclient \
    && for binPath in `ls /usr/bin/ss-* /usr/local/bin/obfs-* /usr/bin/kcp* /usr/bin/v2ray*`; do \
            setcap CAP_NET_BIND_SERVICE=+eip $binPath; \
       done \
    && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && adduser -h /tmp -s /sbin/nologin -S -D -H shadowsocks \
    && adduser -h /tmp -s /sbin/nologin -S -D -H kcptun \
    && apk del .build-deps .build-deps-kernel \
    && apk add --no-cache \
      $(scanelf --needed --nobanner /usr/bin/ss-* /usr/local/bin/obfs-* \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u) \
    && rm -rf /linux-headers-4.19.36-r0.apk \
        kcptun-linux-arm64-${KCP_VERSION}.tar.gz \
        shadowsocks-libev \
        simple-obfs \
        v2ray_plugin.tar.gz \
        /etc/service \
        /var/cache/apk/* \
        /v2ray-plugin_linux_arm*

SHELL ["/bin/bash"]

COPY runit /etc/service
COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
