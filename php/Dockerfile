FROM php:8.2-fpm-alpine as app

# Useful PHP extension installer image, copy binary into your container
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin

# RUN docker-php-ext-install pdo pdo_mysql
RUN set -eux; \
  install-php-extensions pdo pdo_mysql

# allow super user - set this if you use composer as a
# super user at all times like in docker containers
ENV COMPOSER_ALLOW_SUPERUSER=1

# obtain composer using multi-stage build
# https://docs.docker.com/build/building/multi-stage
COPY --from=composer:2.4 /usr/bin/composer /usr/bin/composer

# Here, we are copying only composer.json and composer.lock (instead of copying the entire source)
# right before doing composer install.
# This is enough to take advantage of docker cache and composer install will
# be executed only when composer.json or composer.lock have indeed changed!-
# https://medium.com/@softius/faster-docker-builds-with-composer-install-b4d2b15d0fff
COPY ./app/composer.* ./

# install
RUN composer install --prefer-dist --no-dev --no-scripts --no-progress --no-interaction

# copy application files to the working directory
COPY ./app .

# run composer dump-autoload --optimize
RUN composer dump-autoload --optimize


# Dev Image
# This stage is meant to be target-built into a separate image
# https://docs.docker.com/develop/develop-images/multistage-build/#stop-at-a-specific-build-stage
# https://docs.docker.com/compose/compose-file/#target
FROM app as app_dev

ENV XDEBUG_MODE=off

COPY ./php/conf.d/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN set -eux; \
  install-php-extensions xdebug
