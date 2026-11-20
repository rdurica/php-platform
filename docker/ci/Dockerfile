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
