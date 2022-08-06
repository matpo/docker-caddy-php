FROM matpo/caddy-php

RUN apk add --no-cache freetype libpng libjpeg-turbo freetype-dev libpng-dev libjpeg-turbo-dev \
    xvfb ttf-freefont fontconfig dbus \
    gmp-dev re2c libmcrypt-dev file zlib-dev curl-dev libxml2-dev imagemagick-dev
RUN ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/local/include/
RUN docker-php-ext-configure gd \
    --enable-gd \
    --with-freetype \
    --with-jpeg && \
    NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
    docker-php-ext-install -j${NPROC} gd && \
    apk del --no-cache freetype-dev libpng-dev libjpeg-turbo-dev

RUN docker-php-ext-configure gmp
RUN docker-php-ext-install gmp
RUN docker-php-ext-install soap

# Install graphicsmagick
RUN apk add --no-cache graphicsmagick
RUN apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/v3.10/main/ jasper-dev
RUN apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS libtool g++ make autoconf graphicsmagick-dev
RUN pecl install gmagick-2.0.5RC1 && docker-php-ext-enable gmagick

RUN apk del --no-cache gmp-dev libmcrypt-dev zlib-dev libxml2-dev

# Install wkhtmltopdf from `testing` repository
RUN apk add qt5-qtbase-dev libmhash-dev \
    wkhtmltopdf \
    --no-cache \
    --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ \
    --allow-untrusted \
    && \
    # Wrapper for xvfb
    mv /usr/bin/wkhtmltopdf /usr/bin/wkhtmltopdf-origin && \
    echo $'#!/usr/bin/env sh\n\
    Xvfb :0 -screen 0 1024x768x24 -ac +extension GLX +render -noreset & \n\
    DISPLAY=:0.0 wkhtmltopdf-origin $@ \n\
    killall Xvfb\
    ' > /usr/bin/wkhtmltopdf && \
    chmod +x /usr/bin/wkhtmltopdf

RUN echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/memory_limit.ini
RUN echo "php_admin_value[memory_limit] = 512M" >> /usr/local/etc/php-fpm.d/zz-docker.conf
