FROM php:7.3-apache

MAINTAINER Wilfried Kakou <wilfried.kakou@smile.ci>
# Environmental Variables
ENV PHP_MEMORY_LIMIT 1024M
ENV PHP_DATE_TIMEZONE UTC

# Download script to install PHP extensions and dependencies
ADD https://raw.githubusercontent.com/mlocati/docker-php-extension-installer/master/install-php-extensions /usr/local/bin/

RUN chmod uga+x /usr/local/bin/install-php-extensions && sync

RUN DEBIAN_FRONTEND=noninteractive apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get install -qq -y \
      curl \
      git \
      zip unzip \
    && install-php-extensions \
      bcmath \
      bz2 \
      calendar \
      exif \
      gd \
      intl \
      ldap \
      memcached \
      mysqli \
      opcache \
      pdo_mysql \
      pdo_pgsql \
      pgsql \
      redis \
      soap \
      xsl \
      zip \
      sockets
# already installed:
#      iconv \
#      mbstring \

# install php-redis

RUN apt-get update && \
    pecl channel-update pecl.php.net && \
    pecl install apcu igbinary mongodb && \
    # compile Redis with igbinary support
    pecl bundle redis && cd redis && phpize && ./configure --enable-redis-igbinary && make && make install && \
    docker-php-ext-install bcmath sockets && \
    docker-php-ext-enable apcu igbinary mongodb opcache redis && \
    docker-php-source delete && \
    rm -r /tmp/* /var/cache/* /var/www/html/*


# Installation de composer
ENV COMPOSER_ALLOW_SUPERUSER=1
RUN curl -sS https://getcomposer.org/installer | php -- --filename=composer --install-dir=/usr/local/bin

# Set timezone
RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/London /etc/localtime
RUN "date"

# Installation de ldap
RUN set -x \
    && apt-get update \
    && apt-get install -y libldap2-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install ldap \
    && apt-get purge -y --auto-remove libldap2-dev

# Memory Limit
RUN echo "memory_limit=${PHP_MEMORY_LIMIT}" > /usr/local/etc/php/conf.d/docker-php-ext-memory-limit.ini

# Time Zone
RUN echo "date.timezone=${PHP_DATE_TIMEZONE}" > /usr/local/etc/php/conf.d/docker-php-ext-date-timezone.ini


# install sendmail package
RUN apt-get update && apt-get install -y sendmail sendmail-bin
# sendmail
RUN echo "SMTP=mailcatcher" > /usr/local/etc/php/conf.d/docker-php-ext-mailcatcher.ini
#RUN echo "smtp_port=25" > /usr/local/etc/php/conf.d/docker-php-ext-sendmail.ini
RUN echo "sendmail_path=/usr/bin/env catchmail --smtp-ip mailcatcher -f docker@symfony.dev" > /usr/local/etc/php/conf.d/docker-php-ext-mailcatcher.ini

#restart  sendmail if not started by default
RUN service sendmail restart

#RUN apt-get update &&  apt-get install -y  ssmtp && \
#    echo "sendmail_path = /usr/sbin/ssmtp -t" > /usr/local/etc/php/conf.d/sendmail.ini && \
#    echo "mailhub=mail:25\nUseTLS=NO\nFromLineOverride=YES" > /etc/ssmtp/ssmtp.conf

#test email sending
#RUN sed -i -r 's/\;ping.path/ping.path/'  /usr/local/etc/php-fpm.d/www.conf \
#    && sed  -r 's/\;ping.response = pong/ping.response=OK/' -i  /usr/local/etc/php-fpm.d/www.conf


# Activation d'Apache mod_rewrite
RUN a2enmod rewrite

# On configure un vhost, pour ne pas avoir de public/ ou web/ dans l'URL
COPY vhost.conf /etc/apache2/sites-enabled/000-default.conf
COPY php.ini /etc/php/7.2/php.ini
COPY php-fpm-pool.conf /etc/php/7.2/pool.d/www.conf
