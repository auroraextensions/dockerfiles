FROM php:7.2-fpm

ENV NAME app
ENV CUID 1000
ENV MAILHOG_FILE mhsendmail_linux_amd64
ENV MAILHOG_LINK "https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/${MAILHOG_FILE}"

RUN PACKAGES=" \
        cron \
        git \
        gzip \
        libfreetype6-dev \
        libicu-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libxslt1-dev \
        lsof \
        mariadb-client \
        vim \
        zip \
    "; \
    set -x && \
    apt-get update && \
    apt-get install -y \
        $PACKAGES

RUN set -x && \
    docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/

RUN EXTENSIONS=" \
        bcmath \
        gd \
        intl \
        mbstring \
        opcache \
        pdo_mysql \
        soap \
        xsl \
        zip \
    "; \
    set -x && \
    docker-php-ext-install \
        $EXTENSIONS

RUN PACKAGE="mcrypt"; \
    set -x && \
    pecl install \
        "${PACKAGE}-1.0.2" && \
    docker-php-ext-enable "${PACKAGE}"

RUN set -x && \
    echo "deb http://deb.debian.org/debian stretch-backports main" \
        >>/etc/apt/sources.list

RUN set -x && \
    apt-get update && \
    apt-get install -y \
        libsodium-dev

RUN set -x && \
    pecl install \
        -f libsodium-1.0.17 && \
    echo "extension=sodium.so" >/usr/local/etc/php/conf.d/sodium.ini

RUN set -x && \
    pecl channel-update pecl.php.net && \
    pecl install xdebug && \
    docker-php-ext-enable xdebug && \
    sed -i -e 's/^zend_extension/\;zend_extension/g' \
        /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

RUN set -x && \
    apt-get install -y libssh2-1-dev && \
    pecl install ssh2-1.1.2 && \
    docker-php-ext-enable ssh2

RUN set -x && \
    curl -sS https://getcomposer.org/installer \
    | php -- \
        --install-dir=/usr/local/bin \
        --filename=composer

RUN set -x && \
    groupadd -g "${CUID}" "${NAME}" && \
    useradd -g "${CUID}" \
        -u "${CUID}" \
        -d /var/www \
        -s /bin/bash \
        "${NAME}"

RUN set -x && \
    apt-get install -y gnupg && \
    curl -fsL https://deb.nodesource.com/setup_8.x | bash - && \
    apt-get install -y nodejs && \
    mkdir -p /var/www/.config /var/www/.npm && \
    chown "${NAME}:${NAME}" \
        /var/www/.config \
        /var/www/.npm && \
    ln -sf /var/www/html/node_modules/grunt/bin/grunt \
        /usr/bin/grunt

RUN set -x && \
    curl -fsSLO "${MAILHOG_LINK}" && \
    chmod +x "${MAILHOG_FILE}" && \
    mv -f "${MAILHOG_FILE}" /usr/local/bin/mhsendmail

RUN set -x && \
    printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/update/cron.php\n' >>/etc/crontab && \
    printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/bin/magento cron:run\n' >>/etc/crontab && \
    printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/bin/magento setup:cron:run\n#\n' >>/etc/crontab

COPY ./bin/croninit /usr/local/bin/croninit
COPY ./etc/www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./etc/php.ini /usr/local/etc/php/php.ini
COPY ./etc/php-fpm.conf /usr/local/etc/php-fpm.conf

RUN set x && \
    mkdir /sock && \
    chown -R "${NAME}:${NAME}" \
        /usr/local/etc/php/conf.d \
        /sock

EXPOSE 9001
USER "${NAME}:${NAME}"

VOLUME /var/www
WORKDIR /var/www/html
