FROM php:8.0-fpm-alpine3.12 AS service_php

RUN apk add --no-cache \
        make \
		acl \
		fcgi \
		file \
		gettext \
		git \
        postgresql-dev \
        gnu-libiconv \
	;

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		icu-dev \
		libzip-dev \
        zlib-dev \
	; \
	\
	docker-php-ext-configure zip; \
	docker-php-ext-install -j$(nproc) \
		intl \
		pdo_mysql \
        pdo_pgsql \
        zip \
	; \
	pecl install \
		apcu \
	; \
	pecl clear-cache; \
	docker-php-ext-enable \
		apcu \
		opcache \
	; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-cache --virtual .api-phpexts-rundeps $runDeps; \
	\
	apk del .build-deps

COPY --from=composer:2.0 /usr/bin/composer /usr/bin/composer

RUN ln -fsn /usr/local/bin/php /usr/bin/php
RUN ln -fsn /usr/local/bin/php-config /usr/bin/php-config

RUN ln -s $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini
COPY .docker/php/conf.d/service.prod.ini $PHP_INI_DIR/conf.d/service.ini

COPY .docker/php/php-fpm.d/zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf

ENV COMPOSER_ALLOW_SUPERUSER=1
ENV PATH="${PATH}:/root/.composer/vendor/bin"

WORKDIR /app

COPY ./classes/ /app/classes/
COPY ./public/ /app/public/
COPY ./composer.json /app/composer.json
COPY ./composer.lock /app/composer.lock

RUN composer install --no-interaction

COPY .docker/php/docker-entrypoint.sh /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint

ENTRYPOINT ["docker-entrypoint"]
CMD ["php-fpm"]
