ARG COMPOSER_VERSION=1
ARG composerPath=composer:$COMPOSER_VERSION
FROM $composerPath as composer

FROM trafex/alpine-nginx-php7:1.10.0

USER root

ENV TZ=Mexico/General
ENV HOME="/home/noroot"
ENV PATH="${HOME}/.composer/vendor/drupal/coder/coder_sniffer:${HOME}/.composer/vendor/drupal/sirbrillig/phpcs-variable-analysis:${HOME}/.composer/vendor/drupal/slevomat/coding-standard:${HOME}/.composer/vendor/bin:${PATH}"

LABEL maintainer="Luis Yair Miranda Gonzalez <contacto@yairmiranda.com>"

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apk update && apk upgrade

RUN apk add --update icu-dev bash git rsync nodejs npm yarn automake git file patch
RUN apk add --update php7-tokenizer php7-pdo php7-simplexml php7-pdo_mysql php7-xdebug php7-dev gcc musl-dev make php7-fileinfo php7-iconv php7-xmlwriter php7-zip mysql-client php7-apcu php7-pecl-uploadprogress

#RUN echo 'zend_extension=xdebug.so' > /etc/php7/conf.d/xdebug.ini

COPY --from=composer /usr/bin/composer /usr/local/bin/composer
COPY ./config/php/memory_limit.ini /etc/php7/conf.d/memory_limit.ini
COPY ./config/nginx/nginx-server.conf /etc/nginx/nginx.conf

RUN curl -OL https://github.com/drush-ops/drush-launcher/releases/latest/download/drush.phar \
    && chmod +x drush.phar \
    && mv drush.phar /usr/local/bin/drush

RUN npm install -g @emulsify/cli gulp-cli

RUN adduser -D -u 1000 noroot
RUN addgroup noroot www-data

RUN mkdir -p /var/www/app

RUN chown -R noroot.noroot /var/www/html && \
  chown -R noroot.noroot /run && \
  chown -R noroot.noroot /var/lib/nginx && \
  chown -R noroot.noroot /var/log/nginx && \
  chown -R noroot.noroot /var/www/app && \
  chown -R noroot.noroot /home/noroot/.npm && \
  chown -R noroot.noroot /home/noroot

USER noroot

WORKDIR /var/www/app