FROM caddy:2.2.1 as caddy

FROM php:7.4-fpm

RUN apt-get update
RUN apt-get install -y autoconf zlib1g-dev
RUN pecl install grpc
RUN docker-php-ext-enable grpc

ENV PHP_EXTENSIONS \
    opcache \
    zip \
    gd \
    bcmath \
    mysqli \
    pdo_mysql \
    pgsql \
    pdo_pgsql \
    xsl

ENV PECL_EXTENSIONS \
    redis

RUN apt-get install -y curl zip unzip libzip-dev
RUN pecl install $PECL_EXTENSIONS \
    && docker-php-ext-install -j "$(nproc)" $PHP_EXTENSIONS \
    && docker-php-ext-enable $PECL_EXTENSIONS \
    && docker-php-source delete

COPY --from=caddy /config /config
COPY --from=caddy /data /data
COPY --from=caddy /usr/bin/caddy /usr/bin/caddy
COPY Caddyfile /etc/caddy/

COPY config $PHP_INI_DIR/conf.d
COPY php-fpm /usr/local/etc/php-fpm.d
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

COPY php-caddy-entrypoint /usr/local/bin/
RUN chmod +x /usr/local/bin/php-caddy-entrypoint
RUN curl --tlsv1 -v -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

#Install node
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get install -y nodejs
RUN apt update && apt install -y --no-install-recommends yarn

EXPOSE 8080


ENTRYPOINT [ "php-caddy-entrypoint" ]

CMD ["caddy", "run", "--config", "/etc/caddy/Caddyfile", "--adapter", "caddyfile"]