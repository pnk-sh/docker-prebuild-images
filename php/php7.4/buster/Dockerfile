FROM php:7.4-fpm-buster
LABEL maintainer="hi@pnk.dk"

RUN apt update && apt dist-upgrade -y
RUN apt install -y \
    zlib1g-dev \
    libicu-dev \
    g++ \
    libxml2-dev \
    libfreetype6-dev \
    libwebp-dev \
    libxpm-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libonig-dev \
    libzip-dev \
    libgd3 \
    libgd-dev

# Install NGINX
RUN apt install nginx supervisor -y

# Install NGINX, PHP-FPM and Supervisord
COPY ./php/deploy-files/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./php/deploy-files/nginx/sites/ /etc/nginx/sites-available
COPY ./php/deploy-files/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./php/deploy-files/php-fpm/www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./php/deploy-files/php-fpm/www1.conf /usr/local/etc/php-fpm.d/www1.conf
COPY ./php/deploy-files/php-fpm/www2.conf /usr/local/etc/php-fpm.d/www2.conf
COPY ./php/deploy-files/php-fpm/php.ini /usr/local/etc/php/php.ini


# php-redis
RUN pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini

# # apcu
# RUN apk add --update --no-cache --virtual .build-dependencies $PHPIZE_DEPS \
#     && pecl install apcu \
#     && docker-php-ext-enable apcu \
#     && pecl clear-cache \
#     && apk del .build-dependencies

# Install php extenions
RUN docker-php-ext-install mbstring
RUN docker-php-ext-configure intl && docker-php-ext-install intl
RUN docker-php-ext-install xmlrpc
RUN docker-php-ext-install gettext
RUN docker-php-ext-install sockets
RUN docker-php-ext-install mysqli pdo_mysql
RUN docker-php-ext-install zip
RUN docker-php-ext-install exif
RUN docker-php-ext-install pcntl

RUN docker-php-ext-configure gd \
    --with-jpeg \
    --with-freetype

RUN docker-php-ext-install gd

RUN docker-php-ext-install opcache && docker-php-ext-enable opcache

# Install WP CLI
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
RUN chmod +x wp-cli.phar
RUN mv wp-cli.phar /usr/local/bin/wp
RUN wp --info

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=1.10.19

RUN mkdir /usr/local/log

EXPOSE 8080

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping