FROM php:8.0-fpm-alpine
LABEL maintainer="hi@pnk.dk"

RUN apk update
RUN apk add build-base bash

# Install NGINX
RUN apk add nginx supervisor

# Install NGINX, PHP-FPM and Supervisord
COPY ./php/deploy-files/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./php/deploy-files/nginx/sites/ /etc/nginx/sites-available
COPY ./php/deploy-files/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./php/deploy-files/php-fpm/www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./php/deploy-files/php-fpm/www1.conf /usr/local/etc/php-fpm.d/www1.conf
COPY ./php/deploy-files/php-fpm/www2.conf /usr/local/etc/php-fpm.d/www2.conf
COPY ./php/deploy-files/php-fpm/php.ini /usr/local/etc/php/php.ini

# Install locals
ENV MUSL_LOCALE_DEPS cmake make musl-dev gcc gettext-dev libintl
RUN apk add --no-cache \
    $MUSL_LOCALE_DEPS \
    && wget https://gitlab.com/rilian-la-te/musl-locales/-/archive/master/musl-locales-master.zip \
    && unzip musl-locales-master.zip \
    && cd musl-locales-master \
    && cmake -DLOCALE_PROFILE=OFF -D CMAKE_INSTALL_PREFIX:PATH=/usr . && make && make install \
    && cd .. && rm -r musl-locales-*

# Add packages
RUN apk add \
    libzip-dev \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl

# Install GD extension
RUN apk add --no-cache freetype libpng libjpeg-turbo freetype-dev libpng-dev libjpeg-turbo-dev && \
    docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg \
    NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
    docker-php-ext-install -j$(nproc) gd && \
    apk del --no-cache freetype-dev libpng-dev libjpeg-turbo-dev

# Install intl extension
RUN apk add --no-cache \
    icu-dev \
    && docker-php-ext-install -j$(nproc) intl \
    && docker-php-ext-enable intl \
    && rm -rf /tmp/*

# Install mbstring extension
RUN apk add --no-cache \
    oniguruma-dev \
    && docker-php-ext-install mbstring \
    && docker-php-ext-enable mbstring \
    && rm -rf /tmp/*

# php-redis
ENV PHPREDIS_VERSION 5.3.2

RUN echo http://dl.alpinelinux.org/alpine/latest-stable/community/ >> /etc/apk/repositories \
    && apk add --no-cache shadow curl vim zlib-dev autoconf g++ make re2c \
    && pecl install -o -f redis \
    && docker-php-ext-enable redis.so \
    && pecl clear-cache

# apcu
RUN apk add --update --no-cache --virtual .build-dependencies $PHPIZE_DEPS \
    && pecl install apcu \
    && docker-php-ext-enable apcu \
    && pecl clear-cache \
    && apk del .build-dependencies

# Install php extenions
RUN docker-php-ext-install sockets
RUN docker-php-ext-install mysqli pdo_mysql
RUN docker-php-ext-install zip
RUN docker-php-ext-install exif
RUN docker-php-ext-install pcntl
RUN docker-php-ext-install opcache && docker-php-ext-enable opcache

# Install composer
RUN export COMPOSER_MEMORY_LIMIT=-1
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=1.10.19
RUN mkdir /usr/local/log

# Cleanup in /var/www folder
RUN rm -rf /var/www/*

EXPOSE 8080

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping