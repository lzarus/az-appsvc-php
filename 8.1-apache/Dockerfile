FROM mcr.microsoft.com/oryx/php:8.1-20220825.1
LABEL maintainer="Azure App Services Container Images <appsvc-images@microsoft.com>"
LABEL maintainer="Update by Hasiniaina Andriatsiory <hasiniaina.andriatsiory@gmail.com>"
LABEL description="This image docker contains : \
php8.1, composer, apache2, cron and more extensions php."

ENV PHP_VERSION 8.1

RUN apt-get update \
   && apt-get install --yes --no-install-recommends \
      lsb-release ca-certificates apt-transport-https software-properties-common gpg \
      libjpeg-dev \
      libpq-dev \
      libmcrypt-dev \
      libldap2-dev \
      libldb-dev \
      libicu-dev \
      libgmp-dev \
      libmagickwand-dev \
      imagemagick \
      lftp \
      libpng-dev \
      zlib1g-dev \
      libpspell-dev \
      libcurl4 \
      libcurl3-dev \ 
      libbz2-dev \ 
      libxml2-dev \
      libz-dev \
      libzip4 \
      libzip-dev \
      libmemcached-dev \
      libreadline-dev \
      openssh-server \
      apt-utils \
      nano  \
      vim \
      unzip \
      zip \
      libonig-dev \
      iproute2 \
      iputils-ping \
      pdftk \
      expect \
      mkisofs \
      curl \
      net-tools \
      dnsutils \
      tcpdump \
      tcptraceroute \
      telnet \
      iproute2 
RUN docker-php-ext-install \
      bcmath \
      gd \
      bz2 \
      calendar \
      ctype \
      curl \
      dom \
      xml \
      zip \
      mysqli \
      soap \
      sockets \
      mbstring
#other Package : gettext  exif ffi fileinfo filter ftp imagick imap  intl opcache pcntl pdo pdo_dblib pdo_firebird  shmop simplexml odbc sodium spl standard sysvmsg sysvsem sysvshm pdo_oci pdo_odbc pdo_pgsql pdo_sqlite pgsql  posix pspellreflection 

RUN pecl install xmlrpc-beta
RUN docker-php-ext-enable xmlrpc
RUN docker-php-ext-enable redis
RUN pecl install memcached

#composer
RUN  curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

#Cron
RUN apt-get -y install cron
RUN touch /var/log/cron.log

COPY tcpping /usr/bin/tcpping
RUN chmod 755 /usr/bin/tcpping

COPY init_container.sh /bin/
COPY hostingstart.html /home/site/wwwroot/hostingstart.html

RUN if [[ "$PHP_VERSION" == "5.6" || "$PHP_VERSION" == "7.0" ]] ; then \
        apt-get install -y libmcrypt-dev \
        && docker-php-ext-install mcrypt; \
    fi
    
RUN chmod 755 /bin/init_container.sh \
    && mkdir -p /home/LogFiles/ \
    && echo "root:Docker!" | chpasswd \
    && echo "cd /home" >> /root/.bashrc \
    && ln -s /home/site/wwwroot /var/www/html \
    && mkdir -p /opt/startup

# configure startup
COPY sshd_config /etc/ssh/
COPY ssh_setup.sh /tmp
RUN mkdir -p /opt/startup \
   && chmod -R +x /opt/startup \
   && chmod -R +x /tmp/ssh_setup.sh \
   && (sleep 1;/tmp/ssh_setup.sh 2>&1 > /dev/null) \
   && rm -rf /tmp/*

COPY startssh.sh /opt/startup
RUN chmod +x /opt/startup/startssh.sh

ENV PORT 8080
ENV SSH_PORT 2222
EXPOSE 2222 8080
COPY sshd_config /etc/ssh/

ENV WEBSITE_ROLE_INSTANCE_ID localRoleInstance
ENV WEBSITE_INSTANCE_ID localInstance
ENV PATH ${PATH}:/home/site/wwwroot

RUN sed -i 's!ErrorLog ${APACHE_LOG_DIR}/error.log!ErrorLog /dev/stderr!g' /etc/apache2/apache2.conf 
RUN sed -i 's!User ${APACHE_RUN_USER}!User www-data!g' /etc/apache2/apache2.conf 
RUN sed -i 's!User ${APACHE_RUN_GROUP}!Group www-data!g' /etc/apache2/apache2.conf 
RUN { \
   echo 'DocumentRoot /home/site/wwwroot'; \
   echo 'DirectoryIndex default.htm default.html index.htm index.html index.php hostingstart.html'; \
   echo 'CustomLog /dev/null combined'; \
   echo '<FilesMatch "\.(?i:ph([[p]?[0-9]*|tm[l]?))$">'; \
   echo '   SetHandler application/x-httpd-php'; \
   echo '</FilesMatch>'; \
   echo '<DirectoryMatch "^/.*/\.git/">'; \
   echo '   Order deny,allow'; \
   echo '   Deny from all'; \
   echo '</DirectoryMatch>'; \
   echo 'EnableMMAP Off'; \
   echo 'EnableSendfile Off'; \
} >> /etc/apache2/apache2.conf

RUN rm -f /usr/local/etc/php/conf.d/php.ini \
   && { \
                echo 'error_log=/dev/stderr'; \
                echo 'display_errors=Off'; \
                echo 'log_errors=On'; \
                echo 'display_startup_errors=Off'; \
                echo 'date.timezone=UTC'; \
                echo 'zend_extension=opcache'; \
    } > /usr/local/etc/php/conf.d/php.ini

RUN rm -f /etc/apache2/conf-enabled/other-vhosts-access-log.conf
RUN rm /etc/apache2/sites-enabled/000-default.conf

COPY mpm_prefork.conf /etc/apache2/mods-available/mpm_prefork.conf

WORKDIR /home/site/wwwroot

ENTRYPOINT ["/bin/init_container.sh"]
