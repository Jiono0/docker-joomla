FROM php:5.6-fpm-alpine
LABEL maintainer="Michael Babker <michael.babker@joomla.org> (@mbabker)"

# Disable remote database security requirements.
ENV JOOMLA_INSTALLATION_DISABLE_LOCALHOST_CHECK=1

# entrypoint.sh dependencies
RUN apk add --no-cache \
	bash

# Install PHP extensions
RUN set -ex; \
	\
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		autoconf \
		bzip2-dev \
		libjpeg-turbo-dev \
		libmcrypt-dev \
		libmemcached-dev \
		libpng-dev \
		openldap-dev \
		pcre-dev \
		postgresql-dev \
	; \
	\
	docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
	docker-php-ext-configure ldap; \
	docker-php-ext-install \
		bz2 \
		gd \
		ldap \
		mcrypt \
		mysqli \
		pdo_mysql \
		pdo_pgsql \
		pgsql \
		zip \
	; \
	\
# pecl will claim success even if one install fails, so we need to perform each install separately
	pecl install APCu-4.0.11; \
	pecl install memcached-2.2.0; \
	pecl install redis-4.1.1; \
	\
	docker-php-ext-enable \
		apcu \
		memcached \
		redis \
	; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
		| tr ',' '\n' \
		| sort -u \
		| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
		)"; \
	apk add --virtual .joomla-phpext-rundeps $runDeps; \
	apk del .build-deps

VOLUME /var/www/html

# Define Joomla version and expected SHA1 signature
ENV JOOMLA_VERSION 3.8.12
ENV JOOMLA_SHA1 a354eb7689fe3077d1a14ba3f8a7327017bafd1b

# Download package and extract to web volume
RUN curl -o joomla.tar.bz2 -SL https://github.com/joomla/joomla-cms/releases/download/${JOOMLA_VERSION}/Joomla_${JOOMLA_VERSION}-Stable-Full_Package.tar.bz2 \
	&& echo "$JOOMLA_SHA1 *joomla.tar.bz2" | sha1sum -c - \
	&& mkdir /usr/src/joomla \
	&& tar -xf joomla.tar.bz2 -C /usr/src/joomla \
	&& rm joomla.tar.bz2 \
	&& chown -R www-data:www-data /usr/src/joomla

# Copy init scripts and custom .htaccess
COPY docker-entrypoint.sh /entrypoint.sh
COPY makedb.php /makedb.php

ENTRYPOINT ["/entrypoint.sh"]
CMD ["php-fpm"]
