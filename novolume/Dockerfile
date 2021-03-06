#
# Nasqueron  - Base nginx / php-fpm image
#

FROM debian:jessie
MAINTAINER Sébastien Santoro aka Dereckson <dereckson+nasqueron-docker@espace-win.org>

#
# Prepare the container
#

ENV PHP_VERSION 5.6.35
ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=app --with-fpm-group=app
ENV PHP_INI_DIR /usr/local/etc/php
ENV PHP_BUILD_DEPS bzip2 \
		file \
		libbz2-dev \
		libcurl4-openssl-dev \
		libjpeg-dev \
		libmcrypt-dev \
		libpng12-dev \
		libreadline6-dev \
		libssl-dev \
		libxslt1-dev \
		libxml2-dev
ENV LANG C.UTF-8

RUN apt-get update && apt-get install -y ca-certificates curl libxml2 autoconf \
    gcc libc-dev make pkg-config nginx-full \
    runit nano less tmux wget git locales \
    $PHP_BUILD_DEPS $PHP_EXTRA_BUILD_DEPS \
    --no-install-recommends && rm -r /var/lib/apt/lists/* \
    && dpkg-reconfigure locales

RUN gpg --keyserver pool.sks-keyservers.net --recv-keys 6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3 0BD78B5F97500D450838F95DFE857D9A90D90EC1 \
	&& mkdir -p $PHP_INI_DIR/conf.d \
	&& set -x \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.bz2/from/this/mirror" -o php.tar.bz2 \
	&& curl -SL "http://php.net/get/php-$PHP_VERSION.tar.bz2.asc/from/this/mirror" -o php.tar.bz2.asc \
	&& gpg --verify php.tar.bz2.asc \
	&& mkdir -p /usr/src/php \
	&& tar -xof php.tar.bz2 -C /usr/src/php --strip-components=1 \
	&& rm php.tar.bz2* \
	&& cd /usr/src/php \
	&& ./configure \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		$PHP_EXTRA_CONFIGURE_ARGS \
		--disable-cgi \
		--enable-mysqlnd \
		--enable-bcmath \
		--with-bz2 \
		--enable-calendar \
		--with-curl \
		--with-gd \
		--with-jpeg-dir \
		--enable-gd-native-ttf \
		--enable-mbstring \
		--with-mcrypt \
		--with-mysqli \
		--with-pdo-mysql \
		--enable-pcntl \
		--with-openssl \
		--with-xsl \
		--with-readline \
		--with-zlib \
		--enable-zip \
	&& make -j"$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps \
	&& make clean \
	&& pecl install APCu-4.0.10 \
	&& cd /opt \
	&& curl -sS https://getcomposer.org/installer | php \
	&& ln -s /opt/composer.phar /usr/local/bin/composer

RUN groupadd -r app -g 433 && \
	mkdir /home/app && \
	mkdir -p /var/wwwroot/default && \
	useradd -u 431 -r -g app -d /home/app -s /usr/sbin/nologin -c "Docker image user for web application" app && \
	chown -R app:app /home/app /var/wwwroot/default && \
	chmod 700 /home/app && \
	chmod 711 /var/wwwroot/default

COPY files /

#
# Docker properties
#

EXPOSE 80
EXPOSE 443

CMD ["/usr/local/sbin/runsvdir-init"]
