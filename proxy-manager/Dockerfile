ARG BUILD_FROM=ghcr.io/hassio-addons/base:17.2.5
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Required for webpack config of NPM
ENV NODE_OPTIONS="--openssl-legacy-provider"

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Copy Python requirements file & patches
COPY requirements.txt /tmp/
COPY patches/*.patch /usr/src/

# Setup base
ARG NGINX_PROXY_MANAGER_VERSION="v2.12.3"
# hadolint ignore=DL3003,DL3042
RUN \
    apk add --no-cache --virtual .build-dependencies \
        build-base \
        git \
        libffi-dev \
        openssl-dev \
        patch \
        python3-dev
RUN \
    apk add --no-cache \
        apache2-utils \
        certbot \
        libcap \
        logrotate \
        nginx-mod-stream \
        nginx \
        openssl \
        py3-pip \
        python3 \
        yarn
RUN \
    pip3 install -r /tmp/requirements.txt
RUN \
    yarn global add modclean
RUN \
    curl -J -L -o /tmp/nginxproxymanager.tar.gz \
        "https://github.com/NginxProxyManager/nginx-proxy-manager/archive/v2.12.3.tar.gz"
RUN \
    mkdir /app
RUN \
    tar zxvf \
        /tmp/nginxproxymanager.tar.gz \
        --strip 1 -C /app
# RUN \
#   for i in /usr/src/*.patch; do \
#        patch -d /app -p 1 < "${i}"; done
RUN \
    cd /app/frontend \
    && yarn install \
    && yarn build \
    && rm -rf node_modules
RUN \
    mkdir -p /opt/nginx-proxy-manager/frontend \
    && cp -r /app/frontend/dist/. /opt/nginx-proxy-manager/frontend/
RUN \
    cd /app/backend \
    && yarn install \
    && rm -rf node_modules \
    && cp -r /app/backend/. /opt/nginx-proxy-manager/
RUN \
    cp -R /app/global/. /opt/nginx-proxy-manager/global/ \
    \
    && cd /opt/nginx-proxy-manager \
    && yarn install \
    && rm -rf /etc/services.d/frontend \
    && rm -rf /opt/nginx-proxy-manager/config \
    \
    && rm -f -r /etc/nginx \
    && cp -r /app/docker/rootfs/etc/nginx /etc/nginx \
    && rm -f /etc/nginx/conf.d/dev.conf \
    \
    && cp /app/docker/rootfs/etc/letsencrypt.ini /etc \
    && cp /app/docker/rootfs/etc/logrotate.d/nginx-proxy-manager /etc/logrotate.d
RUN \
    sed -i "s#user npm;#user root;#" /etc/nginx/nginx.conf \
    && sed -i "s#root /app/frontend;#root /opt/nginx-proxy-manager/frontend;#" \
        /etc/nginx/conf.d/production.conf \
    && sed -i "s#table.string('id').notNull().primary();#table.string('id', 32).notNull().primary();#" \
        /opt/nginx-proxy-manager/migrations/20190227065017_settings.js \
    \
    && mkdir -p \
        /run/nginx
RUN \
    modclean \
        --path /opt/nginx-proxy-manager \
        --no-progress \
        --keep-empty \
        --run
RUN \
    yarn global remove modclean \
    && yarn cache clean
RUN \
    apk del --purge .build-dependencies
RUN \
    rm -f -r \
        /app \
        /etc/init.d/nginx \
        /etc/logrotate.d/nginx \
        /opt/nginx-proxy-manager/node_modules/bcrypt/build \
        /root/.cache \
        /root/.node-gyp \
        /tmp/.[!.]* \
        /tmp/* \
        /usr/lib/node_modules \
        /usr/local/share/.cache \
        /var/lib/nginx \
        /var/log/nginx \
        /var/tmp/nginx \
        /var/www

# Copy root filesystem
COPY rootfs /

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
