FROM php:7.2-fpm-buster

ARG _UID=1000
ARG _GID=1000
RUN groupadd -g ${_GID} app || true \
    && useradd -g "$(getent group $_GID | cut -d: -f1)" -u ${_UID} -d /var/www -s /bin/bash app

RUN apt-get update && apt-get install -y \
    cron \
    default-mysql-client \
    git \
    gnupg \
    gzip \
    libbz2-dev \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libmagickwand-dev \
    libmcrypt-dev \
    libonig-dev \
    libpng-dev \
    libsodium-dev \
    libssh2-1-dev \
    libwebp-dev \
    libxslt1-dev \
    libzip-dev \
    lsof \
    mailutils \
    msmtp \
    nodejs \
    procps \
    vim \
    zip \
    wget \
  && rm -rf /var/lib/apt/lists/*

ARG XDEBUG_VERSION=""
RUN pecl channel-update pecl.php.net && pecl install \
    imagick \
    ssh2-1.2 \
    xdebug${XDEBUG_VERSION} \
  && pecl clear-cache \
  && rm -rf /tmp/pear

RUN docker-php-ext-configure \
    gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-webp-dir=/usr/include/ \
  && docker-php-ext-install \
    bcmath \
    bz2 \
    calendar \
    exif \
    gd \
    gettext \
    intl \
    mbstring \
    mysqli \
    opcache \
    pcntl \
    pdo_mysql \
    soap \
    sockets \
    sodium \
    sysvmsg \
    sysvsem \
    sysvshm \
    xsl \
    zip \
  && docker-php-ext-enable \
    imagick \
    ssh2 \
    xdebug

COPY conf/msmtprc /etc/msmtprc

COPY conf/www.conf /usr/local/etc/php-fpm.d/
COPY conf/php.ini /usr/local/etc/php/

ARG XDEBUG_PORT=9003
ENV PATH="${PATH}:/var/www/html"
ENV CHECK_XDEBUG=$XDEBUG_VERSION

RUN sed -i -e "s/listen\.group\s*=\s*app/listen.group = $(getent group $_GID | cut -d: -f1)/g" /usr/local/etc/php-fpm.conf

RUN if [ "$(echo "$CHECK_XDEBUG" | awk -F '' '{print $2}')" = "2" ] ; then echo "\nxdebug.remote_port = ${XDEBUG_PORT}" | tee -a /usr/local/etc/php/php.ini; echo "xdebug.remote_enable = 0" | tee -a /usr/local/etc/php/php.ini; echo "xdebug.remote_host = host.docker.internal" | tee -a /usr/local/etc/php/php.ini; echo "xdebug.remote_autostart = 1" | tee -a /usr/local/etc/php/php.ini; else echo "\nxdebug.client_port = ${XDEBUG_PORT}" | tee -a /usr/local/etc/php/php.ini; echo "xdebug.mode = off" | tee -a /usr/local/etc/php/php.ini; echo "xdebug.client_host = host.docker.internal" | tee -a /usr/local/etc/php/php.ini; echo "xdebug.start_with_request = yes" | tee -a /usr/local/etc/php/php.ini; echo "xdebug.max_nesting_level = 512" | tee -a /usr/local/etc/php/php.ini; fi

RUN mkdir -p /etc/nginx/html /var/www/html /sock \
    && chown -R app /etc/nginx /var/www /usr/local/etc/php/conf.d /sock

USER app

VOLUME /var/www

WORKDIR /var/www/html

EXPOSE 9000



