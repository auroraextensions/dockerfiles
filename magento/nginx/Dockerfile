FROM nginx:1.13

ENV NAME app
ENV CUID 1000

RUN set -x && \
    groupadd -g "${CUID}" "${NAME}" && \
    useradd -g "${CUID}" \
            -u "${CUID}" \
            -d /var/www \
            -s /bin/bash \
            "${NAME}" && \
    touch /var/run/nginx.pid && \
    mkdir -p /etc/nginx/incl.d \
             /sock

RUN PACKAGES=" \
        openssl \
        vim \
    "; \
    set -x && \
    apt-get update && \
    apt-get install -qqy \
        $PACKAGES && \
    mkdir -p /etc/ssl/public \
             /etc/ssl/private && \
    echo -e "\n\n\n\n\n\n\n" \
    | openssl req \
              -x509 \
              -nodes \
              -days 365 \
              -newkey rsa:2048 \
              -keyout /etc/ssl/private/magento.key \
              -out /etc/ssl/public/magento.crt

RUN set -x && \
    chown -R "${NAME}:${NAME}" \
        /etc/ssl/public \
        /etc/ssl/private

COPY ./bin/entrypoint.sh /usr/sbin/entrypoint.sh

COPY ./etc/nginx.conf /etc/nginx/nginx.conf
COPY ./etc/default.conf /etc/nginx/conf.d/default.conf
COPY ./etc/magento.conf /etc/nginx/incl.d/magento.conf

RUN set -x && \
    mkdir -p /etc/nginx/html \
             /var/www/html && \
    chown -R "${NAME}:${NAME}" \
             /etc/nginx \
             /var/www \
             /var/cache/nginx \
             /var/run/nginx.pid \
             /sock

EXPOSE 8080 8443
USER "${NAME}:${NAME}"

VOLUME /var/www
WORKDIR /var/www/html

ENTRYPOINT ["entrypoint.sh"]
CMD ["nginx"]
