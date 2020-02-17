FROM php:7.2-fpm

ENV NAME app
ENV CUID 1000
ENV PATH "${PATH}:/var/www/html/bin:/var/www/html/vendor/bin"
ENV PECL pecl.php.net

RUN PKGS=" \
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
        $PKGS

RUN DIR="/usr/include"; \
    PKG="gd"; \
    set -x && \
    docker-php-ext-configure \
        "${PKG}" \
        --with-freetype-dir="${DIR}" \
        --with-jpeg-dir="${DIR}"

RUN EXTS=" \
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
        $EXTS

RUN PKG="mcrypt"; \
    set -x && \
    pecl install \
        "${PKG}-1.0.2" && \
    docker-php-ext-enable \
        "${PKG}"

RUN EXT="sodium"; \
    LIB="lib${EXT}"; \
    set -x && \
    apt-get update && \
    apt-get install -y \
        "${LIB}-dev" && \
    pecl install -f \
        "${LIB}-1.0.17"

RUN EXT="xdebug"; \
    set -x && \
    pecl channel-update \
        "${PECL}" && \
    pecl install \
        "${EXT}" && \
    docker-php-ext-enable \
        "${EXT}" && \
    sed -i -e 's/^zend_extension/\;zend_extension/g' \
        /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

RUN EXT="ssh2"; \
    LIB="lib${EXT}"; \
    PKG="${LIB}-1-dev"; \
    set -x && \
    apt-get install -y \
        "${PKG}" && \
    pecl install \
        "${EXT}-1.1.2" && \
    docker-php-ext-enable \
        "${EXT}"

RUN DIR="/usr/local/bin"; \
    set -x && \
    curl -sS https://getcomposer.org/installer \
    | php -- \
        --install-dir="${DIR}" \
        --filename=composer

RUN set -x && \
    groupadd -g "${CUID}" "${NAME}" && \
    useradd -g "${CUID}" \
        -u "${CUID}" \
        -d /var/www \
        -s /bin/bash \
        "${NAME}"

RUN PKG="gnupg"; \
    set -x && \
    apt-get install -y \
        "${PKG}"

RUN PKG="nodejs"; \
    set -x && \
    curl -fsL https://deb.nodesource.com/setup_8.x | bash - && \
    apt-get install -y \
        "${PKG}" && \
    mkdir -p /var/www/.config \
             /var/www/.npm && \
    chown "${NAME}:${NAME}" \
        /var/www/.config \
        /var/www/.npm && \
    ln -sf /var/www/html/node_modules/grunt/bin/grunt \
           /usr/bin/grunt

RUN FILE="mhsendmail"; \
    EXEC="${FILE}_linux_amd64"; \
    LINK="https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/${EXEC}"; \
    set -x && \
    curl -fsLO "${LINK}" && \
    chmod +x "${EXEC}" && \
    mv -f "${EXEC}" \
          "/usr/local/bin/${FILE}"

RUN set -x && \
    echo "alias ll='ls -laF --color=auto'" >> "/var/www/html/.bashrc" && \
    printf '* * * * * %s /usr/local/bin/php /var/www/html/update/cron.php\n' "${NAME}" >>/etc/crontab && \
    printf '* * * * * %s /usr/local/bin/php /var/www/html/bin/magento cron:run\n' "${NAME}" >>/etc/crontab && \
    printf '* * * * * %s /usr/local/bin/php /var/www/html/bin/magento setup:cron:run\n#\n' "${NAME}" >>/etc/crontab

COPY ./bin/croninit /usr/local/bin/croninit
COPY ./etc/php.ini /usr/local/etc/php/php.ini
COPY ./etc/php-fpm.conf /usr/local/etc/php-fpm.conf
COPY ./etc/www.conf /usr/local/etc/php-fpm.d/www.conf

RUN set x && \
    mkdir -p /etc/nginx/html \
             /var/www/html \
             /sock && \
    chown -R "${NAME}:${NAME}" \
             /etc/nginx \
             /var/www \
             /usr/local/etc/php/conf.d \
             /sock

EXPOSE 9001
USER "${NAME}:${NAME}"

VOLUME /var/www
WORKDIR /var/www/html
