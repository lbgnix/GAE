FROM php:7.4-apache-buster

# install the PHP extensions we need
RUN set -eux; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
		a2enmod headers; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libzip-dev \
	; \
	\
	#For php7.3 or later the --with-freetype-dir becomes --with-freetype and --with-jpeg-dir becomes --with-jpeg. --with-png-dir would not be required.
	docker-php-ext-configure gd \
		--with-freetype=/usr/include \
		--with-jpeg=/usr/include \
	; \
	\
	docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

COPY --from=composer:1.10 /usr/bin/composer /usr/local/bin/

# https://www.drupal.org/node/3060/release
WORKDIR /opt/drupal
RUN rm -rf /opt/drupal/web
COPY . .
COPY php.ini /etc/
RUN set -eux; \
	export COMPOSER_HOME="$(mktemp -d)"; \
	composer install; \
	chown -R www-data:www-data web/sites web/modules web/themes; \
	rmdir /var/www/html; \
        ln -sf /opt/drupal/web /var/www/html; \
	# delete composer cache
	rm -rf "$COMPOSER_HOME"


#RUN sed -i -e '$ r insert.txt' php.ini

RUN sed -i 's/Listen 80/Listen 8080/g' /etc/apache2/ports.conf
RUN apt-get update && apt-get install -y libcurl4-gnutls-dev:i386; \
    apt-get install -y modsecurity-crs
RUN rm -rf /etc/apache2/conf-available/security.conf; \
    rm -rf /etc/apache2/apache2.conf
COPY security.conf /etc/apache2/conf-available/
COPY php.ini /usr/local/etc/php/
COPY apache2.conf /etc/apache2/
COPY 000-default.conf /etc/apache2/sites-enabled/

ENV PATH=${PATH}:/opt/drupal/vendor/bin

# vim:set ft=dockerfile:
