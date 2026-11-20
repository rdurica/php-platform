ARG FRANKENPHP_IMAGE=dunglas/frankenphp:1.12-php8.5-alpine
# hadolint ignore=DL3006
FROM ${FRANKENPHP_IMAGE}

ENV TERM=xterm-256color
ENV TZ=Europe/Prague
ENV SHELL=/bin/bash
ENV COMPOSER_HOME=/usr/local/share/composer
ENV PATH="${COMPOSER_HOME}/vendor/bin:${PATH}"

# System deps + PHP extensions + timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && apk add --no-cache \
        tzdata nodejs npm bash curl git openssh-client \
        linux-headers libpq-dev libzip-dev zlib-dev \
        libxml2-dev icu-dev freetype-dev \
        libjpeg-turbo-dev libpng-dev libwebp-dev \
    && install-php-extensions \
        pdo pdo_mysql mysqli pdo_pgsql pgsql \
        zip intl bcmath gd sockets xdebug

# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Symfony CLI + Laravel installer
RUN curl -sS https://get.symfony.com/cli/installer | bash -s -- --install-dir=/usr/local/bin \
    && mv /usr/local/bin/symfony* /usr/local/bin/symfony 2>/dev/null || true \
    && chmod +x /usr/local/bin/symfony \
    && mkdir -p "${COMPOSER_HOME}" \
    && composer global require laravel/installer \
    && chmod -R a+rX "${COMPOSER_HOME}"

# User setup + aliases
RUN adduser -D -u 1000 -s /bin/bash robbyte \
    && mkdir -p /app/src/node_modules \
    && chown -R robbyte:robbyte /app /data /config \
    && { \
        echo 'alias ll="ls -lah"'; \
        echo 'alias c="composer"'; \
        echo 'alias art="php artisan"'; \
        echo 'alias sf="php bin/console"'; \
        echo 'alias laravel="composer create-project laravel/laravel ."'; \
        echo 'alias nette="composer create-project nette/web-project ."'; \
        echo "PS1='\e[92m\u\e[0m@\e[94m\h\e[0m:\e[35m\w\e[0m# '"; \
    } >> /home/robbyte/.bashrc \
    && chown robbyte:robbyte /home/robbyte/.bashrc

# Default dev configs baked into image
COPY docker/dev/Caddyfile /etc/caddy/Caddyfile
COPY docker/dev/php.ini "$PHP_INI_DIR/conf.d/99-custom.ini"
COPY docker/dev/xdebug.ini "$PHP_INI_DIR/conf.d/xdebug.ini"
COPY --chmod=755 --chown=robbyte:robbyte docker/dev/entrypoint.sh /usr/local/bin/entrypoint.sh

WORKDIR /app/src
USER robbyte

EXPOSE 80 443

CMD ["/usr/local/bin/entrypoint.sh"]
