FROM php:8.0-fpm

USER root

ARG USER_ID
ARG GROUP_ID

RUN userdel -f www-data
RUN if getent group www-data ; then groupdel www-data; fi
RUN groupadd -g ${GROUP_ID} www-data
RUN useradd -l -u ${USER_ID} -g www-data www-data
RUN chown -R www-data:www-data /var/www

RUN install -d -m 0755 -o www-data -g www-data /home/www-data
RUN chown -R www-data:www-data /home/www-data

RUN mkdir /var/log/supervisor
RUN chown -R www-data:www-data /var/log/supervisor
RUN touch /var/run/supervisord.pid
RUN touch /var/run/crond.pid
RUN chown -R www-data:www-data /var/run
RUN chown www-data:www-data /var/run/supervisord.pid /var/run/crond.pid

RUN apt-get update && apt-get install -y vim zip unzip cron supervisor libmcrypt-dev default-mysql-client openssh-client zlib1g-dev ssh libfreetype6-dev libjpeg62-turbo-dev \
    libpng-dev libicu-dev g++ libmagickwand-dev libzip-dev

RUN export EDITOR='/usr/bin/vim'

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN docker-php-ext-configure intl

RUN docker-php-ext-configure gd --with-freetype --with-jpeg

RUN docker-php-ext-install pdo_mysql zip gd exif intl

RUN docker-php-ext-configure intl

RUN docker-php-ext-install pcntl

RUN pecl install redis

RUN docker-php-ext-enable redis

ADD ./docker/php/php.ini /usr/local/etc/php/conf.d/php.ini

ADD ./docker/php/supervisor.conf /etc/supervisor/conf.d/supervisord.conf

ADD ./docker/php/crontab /etc/cron.d/lumen-cron
RUN crontab -u www-data /etc/cron.d/lumen-cron

RUN usermod -a -G crontab www-data
RUN chmod gu+s /usr/sbin/cron
RUN python -c "import socket as s; sock = s.socket(s.AF_UNIX); sock.bind('/dev/shm/supervisor.sock')"

USER www-data

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

WORKDIR /var/www
