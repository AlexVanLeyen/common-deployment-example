FROM alpine:3 AS code-base

ARG TOKEN
ENV GIT_TOKEN=${TOKEN}

RUN apk update \
    && apk add git \
    && apk add openssh \
    && apk add dos2unix

RUN git config --global url."https://${GIT_TOKEN}@github.com/".insteadOf "https://github.com/"

RUN git clone --progress https://github.com/smartnd/outsmart.git > /dev/null


FROM php:7.4.3-apache

ARG SECRETS_BUCKET_NAME

ENV GIT_NAME=dummy
ENV GIT_EMAIL=dummy@example.com
ENV SECRETS_BUCKET_NAME=${SECRETS_BUCKET_NAME}

RUN apt-get update -y \
    && apt-get install -y libpng-dev \
    && apt-get install -y unzip \
    && apt-get install -y dos2unix \
    && apt-get install -y git \
    && apt-get install -y vim \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install gd \
    # && apt-get install -y wkhtmltopdf \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


# PHP Package Manager
RUN  curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer 

# PHP Configurations
COPY ./php.ini /usr/local/etc/php/php.ini

# Apache Configurations
COPY ./mime.types /etc/mime.types

RUN rm -f /etc/apache2/sites-available/000-default.conf
COPY ./apache.config /etc/apache2/sites-available/000-default.conf

# Debugging
RUN pecl install xdebug-2.9.8 \
 && docker-php-ext-enable xdebug \
 && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
 && echo "xdebug.remote_autostart=on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
 && echo "xdebug.remote_connect_back=off" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
 && echo "xdebug.remote_mode=req" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
 && echo "xdebug.remote_host=localhost" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Miscs
# This folder is referenced in the docker-entrypoint script
RUN mkdir /root/.init

# Installing Application
WORKDIR /var/www/html/

COPY --from=code-base /outsmart .

# Installing Application Dependencies
RUN composer install

COPY ./Moneris-eCommerce-Unified-API-PHP /var/www/html/vendor/Moneris-eCommerce-Unified-API-PHP

# Upload Folders
RUN mkdir -p /data/upload/dispensary/item_images \
    && mkdir -p /data/user_form_backup \
    && mkdir -p /data/patient_profile_pic \
    && mkdir -p /data/chart_images

# Upload Folders Soft Links
RUN ln -s /data/upload /var/www/html/upload \
    && ln -s /data/patient_profile_pic /var/www/html/patient_profile_pic \
    && ln -s /data/chart_images /var/www/html/chart_images \
    && ln -s /data/user_form_backup /var/www/html/user_form_backup

# Folders Permissions
RUN chown -R www-data:www-data /var/www/html/ \
    && chown -R www-data:www-data /data/*

# Container Start Script
COPY ./docker-entrypoint /usr/local/bin/docker-entrypoint
RUN dos2unix /usr/local/bin/docker-entrypoint
RUN ["chmod", "+x", "/usr/local/bin/docker-entrypoint"]

# Apache Modules
RUN a2enmod rewrite

# User Bash Settings
RUN rm -f /root/.bashrc
COPY ./bashrc /root/.bashrc

ENTRYPOINT [ "/usr/local/bin/docker-entrypoint" ]

CMD ["apache2-foreground"]
