#
# 1 - Setup composer image
#

FROM composer as php-deps-installer
COPY ./composer.json /app
RUN composer install --no-interaction --ignore-platform-reqs --no-scripts

#
# 2 - Setup php and apache server
#

FROM php:8.2-apache as base
RUN apt-get update

# postgres & php compat
RUN apt-get install -y libpq-dev
RUN docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql
RUN docker-php-ext-install pdo pdo_pgsql pgsql

# prepare Apache server with rewrite rules
COPY ./apache/apache2.conf /etc/apache2/apache2.conf
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf \
    && a2enmod rewrite \
    && a2enmod actions \
    && service apache2 restart

# copy php dependencies from composer image
COPY --from=php-deps-installer /app/vendor /var/www/html/vendor

# set image working directory to server root and clone local dir there
WORKDIR /var/www/html
COPY . .

# start the server
EXPOSE 80
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
