FROM debian:buster-slim AS builder

RUN apt-get update && \
    apt-get install -y build-essential git

RUN mkdir /code && \
    git clone https://github.com/g4klx/YSFClients.git /code && \
    cd /code/YSFReflector/ && \
    make clean all

FROM debian:buster-slim

ENV REFLECTOR_NAME set_me
ENV REFLECTOR_DESCRIPTION set_me
ENV REFLECTOR_HOST set_me

RUN apt-get update && \
    apt-get install -y procps && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    mkdir /app

COPY --from=builder /code/YSFReflector/YSFReflector.ini /app/YSFReflector.ini
COPY --from=builder /code/YSFReflector/YSFReflector /app/YSFReflector
COPY config.php /var/www/html/config/config.php
COPY entrypoint.sh /entrypoint.sh

EXPOSE 42000/udp
EXPOSE 80/tcp

# Install YSF dashboard
ENV APACHE_RUN_DIR /var/lib/apache/runtime
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE=/var/run/apache2-ysfdashboard.pid

RUN apt-get update && \
    mkdir -p ${APACHE_RUN_DIR} && \
    apt-get install -y git && \
    apt-get install -y apache2 && \
    echo "ServerName ${REFLECTOR_HOST}" >> /etc/apache2/apache2.conf && \
    usermod -G www-data -a root && \
    chown -R www-data:www-data /var/www/html && \
    chmod -R 775 /var/www/html && \
    apt install php7.3-common php7.3-cgi php7.3 -y && \
    apt install libapache2-mod-php7.3 -y && \
    cd ~ && \
    git clone https://github.com/dg9vh/YSFReflector-Dashboard.git && \
    cp -R ./YSFReflector-Dashboard/* /var/www/html/ && \
    rm /var/www/html/setup.php && \
    rm /var/www/html/index.html

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/app/YSFReflector", "/app/YSFReflector.ini"]
HEALTHCHECK CMD ps aux | grep [Y]SFReflector || exit 1
