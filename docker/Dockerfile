ARG VERSION_PHP=8.3
ARG VERSION_COMPOSER=lts

FROM composer:${VERSION_COMPOSER} AS comp
FROM php:${VERSION_PHP}-apache

# Labels
LABEL org.opencontainers.image.title="LibreBooking"
LABEL org.opencontainers.image.description="LibreBooking as a container"
LABEL org.opencontainers.image.url="https://github.com/librebooking/docker"
LABEL org.opencontainers.image.source="https://github.com/librebooking/docker"
LABEL org.opencontainers.image.licenses="GPL-3.0"
LABEL org.opencontainers.image.authors="colisee@hotmail.com"

# Copy entrypoint scripts и cron
COPY --chmod=755 ./docker/bin /usr/local/bin/
COPY --chown=www-data:www-data ./docker/lb-jobs-cron /config/

# Copy composer
COPY --from=comp /usr/bin/composer /usr/bin/composer

# Update and install required debian packages
ENV DEBIAN_FRONTEND=noninteractive
RUN set -ex; \
    apt-get update; \
    apt-get upgrade --yes; \
    apt-get install --yes --no-install-recommends \
      cron \
      git \
      libjpeg-dev \
      libldap-dev \
      libpng-dev \
      libfreetype6-dev \
      unzip; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# Customize Apache & PHP + DocumentRoot fix (всё ещё root)
RUN set -ex; \
    chown www-data:www-data /config; \
    cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"; \
    { \
     echo 'RemoteIPHeader X-Real-IP'; \
     echo 'RemoteIPInternalProxy 10.0.0.0/8'; \
     echo 'RemoteIPInternalProxy 172.16.0.0/12'; \
     echo 'RemoteIPInternalProxy 192.168.0.0/16'; \
    } > /etc/apache2/conf-available/remoteip.conf; \
    a2enconf remoteip; \
    a2enmod rewrite headers remoteip; \
    docker-php-ext-configure gd --with-jpeg --with-freetype; \
    docker-php-ext-install mysqli gd ldap; \
    pecl install timezonedb; \
    docker-php-ext-enable timezonedb; \
    mkdir -p /var/log/librebooking; \
    chown www-data:www-data /var/log/librebooking; \
    touch /usr/local/etc/php/conf.d/librebooking.ini; \
    chown www-data:www-data /usr/local/etc/php/conf.d/librebooking.ini; 
    
# Copy local source code
USER www-data
WORKDIR /var/www/html
COPY --chown=www-data:www-data ./src ./

# Install dependencies + fixes + tpl_c (уже от www-data — права на vendor/ будут правильные)
RUN set -ex; \
    if [ -f composer.json ]; then \
      sed -i -e "s:\(.*\)nickdnk/graph-sdk\(.*\)7\.0\(.*\):\1joelbutcher/facebook-graph-sdk\26\.1\3:" composer.json; \
      composer install --no-interaction --no-dev --prefer-dist --optimize-autoloader --no-progress; \
    fi; \
    mkdir -p tpl_c; \
    chown www-data:www-data tpl_c

# Environment
WORKDIR /
VOLUME /config
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["apache2-foreground"]