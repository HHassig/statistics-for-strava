FROM ghcr.io/linuxserver/baseimage-alpine:3.21

# Setup document root
WORKDIR /var/www

# Install packages including composer
RUN apk add --no-cache \
  bash \
  curl \
  nginx \
  php84 \
  php84-bcmath \
  php84-ctype \
  php84-curl \
  php84-dom \
  php84-fileinfo \
  php84-fpm \
  php84-gd \
  php84-iconv \
  php84-intl \
  php84-mbstring \
  php84-opcache \
  php84-openssl \
  php84-pdo \
  php84-pdo_sqlite \
  php84-phar \
  php84-session \
  php84-simplexml \
  php84-tokenizer \
  php84-xml \
  php84-xmlreader \
  php84-xmlwriter \
  php84-zip \
  composer

# Make sure PHP 8.4 is the default and properly configured
RUN ln -sf /usr/bin/php84 /usr/bin/php && \
    ln -sf /etc/php84 /etc/php && \
    php84 --version && \
    php84 -m | grep -E "(bcmath|ctype|dom|simplexml|fileinfo|xml|iconv)" && \
    sed -i 's|/usr/bin/php83|/usr/bin/php84|g' /usr/bin/composer

# Configure nginx - http
COPY docker/app/config/nginx.conf /etc/nginx/nginx.conf

# Configure nginx - default server
COPY docker/app/config/conf.d /etc/nginx/conf.d/

# Configure PHP-FPM
ENV PHP_INI_DIR="/etc/php84"
COPY docker/app/config/fpm-pool.conf ${PHP_INI_DIR}/php-fpm.d/www.conf
COPY docker/app/config/php.ini ${PHP_INI_DIR}/conf.d/custom.ini

COPY docker/root /

# Copy composer files and bin/ directory first (for better Docker layer caching)
COPY composer.json composer.lock bin/ /var/www/

# Install PHP dependencies BEFORE copying the rest of the application
RUN composer install --no-dev --optimize-autoloader --no-interaction

# Now copy the rest of the application (excluding vendor if it exists locally)
COPY --chown=abc:abc . /var/www/

# Ensure vendor directory exists and has correct permissions
RUN chown -R abc:abc /var/www/vendor

# Application setup
RUN touch /var/www/.env
RUN rm -Rf docker
RUN mkdir -p /var/www/var/cache/dev /var/www/var/log
RUN chown -R abc:abc /var/www/var/cache/dev /var/www/var/log

# Keep uid/gid same as pre-lsio for backwards compatibility
ENV PUID=65534
ENV PGID=100

# Expose the port nginx is reachable on
EXPOSE 8080

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping || exit 1
