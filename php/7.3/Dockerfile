ARG COMPOSER_VERSION=1

FROM ghcr.io/ymiranda-web/docker-templates:7.3-c$COMPOSER_VERSION-base as image-composer
FROM image-composer

ENV TZ=Mexico/General
ENV HOME="/home/noroot"
ENV PATH="${HOME}/.composer/vendor/drupal/coder/coder_sniffer:${HOME}/.composer/vendor/drupal/sirbrillig/phpcs-variable-analysis:${HOME}/.composer/vendor/drupal/slevomat/coding-standard:${HOME}/.composer/vendor/bin:${PATH}"

LABEL Maintainer="Luis Yair Miranda Gonzalez <yair.miranda@tec.mx>"

USER root

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# musl-dev icu-dev automake
RUN apk add --no-cache bash mysql-client git rsync file patch

RUN apk add --no-cache --virtual .build-deps pcre-dev ${PHPIZE_DEPS} \
    && pecl install uploadprogress \
    && pecl install xdebug \
    && docker-php-ext-enable uploadprogress xdebug ; \
    \ runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-network --virtual .drupal-phpexts-rundeps $runDeps; \
    apk del --no-network .build-deps

COPY config/php/memory_limit.ini /usr/local/etc/php/conf.d/memory_limit.ini

USER noroot