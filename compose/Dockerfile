#
# https://syspass.org
# https://doc.syspass.org
#

FROM php:7.3-apache-stretch

LABEL maintainer=nuxsmin@syspass.org version=3.1.2 php=7.3

RUN apt-get update \
	&& apt-get install -y \
    locales \
    git \
    gosu \
    libicu-dev \
    libldb-dev \
    libldap2-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    unzip \
    ssl-cert \
	&& apt-get clean \
	&& rm -r /var/lib/apt/lists/* \
	&& pecl install xdebug-2.7.1 \
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
	&& docker-php-ext-install -j$(nproc) ldap intl gettext pdo_mysql opcache gd \
	&& docker-php-ext-enable ldap xdebug intl pdo_mysql

ENV APACHE_RUN_USER="www-data" \
    SYSPASS_DIR="/var/www/html/sysPass" \
    SYSPASS_UID=9001 \
    SYSPASS_DEV=0

RUN mkdir -p ${SYSPASS_DIR}

WORKDIR ${SYSPASS_DIR}

LABEL build=19030701

# Custom sysPass Apache config with SSL by default
COPY ["compose/syspass.conf", "/etc/apache2/sites-available/"]

# Xdebug module config
#COPY compose/xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Custom entrypoint
COPY compose/entrypoint.sh /usr/local/sbin/
COPY compose/init-functions /usr/local/sbin/

RUN chmod 755 /usr/local/sbin/entrypoint.sh \
  && a2dissite 000-default default-ssl \
  && a2ensite syspass \
  && a2enmod proxy_fcgi setenvif ssl rewrite \
  && ln -sf /dev/stdout ${APACHE_LOG_DIR}/access.log \
  && ln -sf /dev/stderr ${APACHE_LOG_DIR}/error.log

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

ADD ./composer.json ${SYSPASS_DIR}

RUN /usr/bin/composer install \
      --ignore-platform-reqs \
      --no-interaction \
      --no-plugins \
      --no-scripts \
      --prefer-dist \
      --optimize-autoloader

EXPOSE 80 443

ENTRYPOINT ["/usr/local/sbin/entrypoint.sh"]

CMD ["apache"]
