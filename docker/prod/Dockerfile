ARG FRANKENPHP_VERSION=1.12
ARG PHP_VERSION=8.5
FROM dunglas/frankenphp:${FRANKENPHP_VERSION}-php${PHP_VERSION}-alpine

ENV TERM=xterm-256color
ENV TZ=Europe/Prague

# System deps + PHP extensions + timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && apk add --no-cache \
        tzdata \
        bash \
        curl \
        netcat-openbsd \
        libpq-dev \
        libzip-dev \
        zlib-dev \
        libxml2-dev \
        icu-dev \
        linux-headers \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libwebp-dev \
    && install-php-extensions \
        pdo pdo_mysql mysqli pdo_pgsql pgsql \
        zip intl bcmath gd sockets

# Enable PHP production settings and Caddy config
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
COPY docker/prod/php.ini "$PHP_INI_DIR/conf.d/99-custom.ini"
COPY docker/prod/opcache-jit.ini "$PHP_INI_DIR/conf.d/opcache-jit.ini"
COPY docker/prod/Caddyfile /etc/caddy/Caddyfile
COPY --chmod=755 docker/prod/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

RUN adduser -D -u 1000 -s /bin/sh robbyte \
    && mkdir -p /app/src \
    && chown -R robbyte:robbyte /app /data /config

WORKDIR /app/src
USER robbyte

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost/up || exit 1

CMD ["/usr/local/bin/docker-entrypoint.sh"]
