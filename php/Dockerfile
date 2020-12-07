FROM php:7.4-fpm-alpine

RUN set -ex && apk --no-cache add postgresql-dev

ADD ./www.conf /usr/local/etc/php-fpm.d/www.conf

RUN docker-php-ext-install pdo pdo_pgsql

RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel

RUN mkdir -p /var/www/html

RUN chown laravel:laravel /var/www/html

WORKDIR /var/www/html