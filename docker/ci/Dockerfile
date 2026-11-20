ARG FRANKENPHP_IMAGE=dunglas/frankenphp:1.12-php8.5-alpine
# hadolint ignore=DL3006
FROM ${FRANKENPHP_IMAGE}

ENV TERM=xterm-256color
ENV TZ=Europe/Prague

# System deps + PHP extensions + timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && apk add --no-cache \
        tzdata \
        bash \
        curl \
        git \
        linux-headers \
        libpq-dev \
        libzip-dev \
        zlib-dev \
        libxml2-dev \
        icu-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libwebp-dev \
    && install-php-extensions \
        pdo pdo_mysql mysqli pdo_pgsql pgsql \
        zip intl bcmath gd sockets

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

RUN adduser -D -u 1000 -s /bin/sh robbyte \
    && mkdir -p /app/src \
    && chown -R robbyte:robbyte /app /data /config

WORKDIR /app/src
USER robbyte

CMD ["php", "-v"]
