ARG COMPOSER_VERSION=1
ARG PHP_VERSION=7.4
ARG composerPath=composer:$COMPOSER_VERSION
ARG phpAlpine=php:$PHP_VERSION-fpm-alpine

FROM $composerPath as composer
FROM $phpAlpine as php-alpine

FROM php-alpine

LABEL Maintainer="Luis Yair Miranda Gonzalez <yair.miranda@tec.mx>"

RUN apk update && apk upgrade

# install the PHP extensions we need
RUN set -eux; \
	\
	apk add --no-cache --virtual .build-deps \
		coreutils \
		libzip-dev \
	; \
	\
	docker-php-ext-install -j "$(nproc)" \
		opcache \
		pdo_mysql \
		zip \
	; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-network --virtual .drupal-phpexts-rundeps $runDeps; \
	apk del --no-network .build-deps

# Add gd extension
RUN if echo "$PHP_VERSION" | grep -v "7.4"; then apk add --no-cache \
    freetype libpng libjpeg-turbo freetype-dev libpng-dev libjpeg-turbo-dev && \
    docker-php-ext-configure gd \
    --with-gd \
    --with-freetype-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ && \
    NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
    docker-php-ext-install -j${NPROC} gd && \
    apk del --no-cache freetype-dev libpng-dev libjpeg-turbo-dev  ; fi

RUN if echo "$PHP_VERSION" | grep -q "7.4"; then apk add --no-cache \
      freetype \
      libpng \
      libjpeg-turbo \
      freetype-dev \
      libpng-dev \
      libjpeg-turbo-dev \
      && docker-php-ext-configure gd \
      --with-freetype \
      --with-jpeg \
      && NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
      docker-php-ext-install -j${NPROC} gd && \
      apk del --no-cache freetype-dev libpng-dev libjpeg-turbo-dev  ; fi

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php

RUN apk add --update nginx supervisor

# Configure nginx
COPY config/nginx/nginx.conf /etc/nginx/nginx.conf

# Configure PHP-FPM
COPY config/php/fpm-pool.conf /usr/local/etc/php-fpm.d/www.conf
COPY config/php/php.ini /usr/local/etc/php/conf.d/custom.ini
COPY config/php/opcache.ini /usr/local/etc/php/conf.d/opcache-recommended.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

COPY --from=composer /usr/bin/composer /usr/local/bin/composer

RUN curl -OL https://github.com/drush-ops/drush-launcher/releases/latest/download/drush.phar \
    && chmod +x drush.phar \
    && mv drush.phar /usr/local/bin/drush

# Setup document root
RUN mkdir -p /var/www/app  && \
    mkdir -p /home/nobody

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /var/www && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/app

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping