FROM php:7.3-fpm

MAINTAINER Patrick McLain <pat@pmclain.com>

# Install dependencies
RUN apt-get update \
  && apt-get install -y \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libxslt1-dev \
    sendmail-bin \
    sendmail \
    sudo \
    libzip-dev \
    wget \
    gnupg2

RUN echo 'deb http://apt.newrelic.com/debian/ newrelic non-free' | tee /etc/apt/sources.list.d/newrelic.list
RUN wget -O- https://download.newrelic.com/548C16BF.gpg | apt-key add -
RUN apt-get update && apt-get -y install newrelic-php5
RUN NR_INSTALL_SILENT=1 newrelic-install install

# Configure the gd library
RUN docker-php-ext-configure \
  gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

# Install required PHP extensions
RUN docker-php-ext-install \
  dom \
  gd \
  intl \
  mbstring \
  pdo_mysql \
  xsl \
  zip \
  soap \
  bcmath \
  pcntl \
  sockets \
  opcache

ENV MAGENTO_ROOT /var/www/magento

RUN echo 'memory_limit=512m' >> /usr/local/etc/php/conf.d/zz-magento.ini
COPY etc/opcache.ini /usr/local/etc/php/conf.d/99-opcache.ini

ADD etc/php-fpm.conf /usr/local/etc/

RUN groupadd nginx
RUN adduser --system --disabled-password nginx
RUN mkdir -p /var/www/vhosts/{{ hostname }}/current
RUN chown nginx:nginx /var/www/vhosts/{{ hostname }}/current

RUN sed -i -e "s/REPLACE_WITH_REAL_KEY/{{ newrelic_key }}/" \
  -e "s/newrelic.appname[[:space:]]=[[:space:]].*/newrelic.appname=\"{{ newrelic_app_name }}\"/" \
  -e '$anewrelic.distributed_tracing_enabled=true' \
  $(php -r "echo(PHP_CONFIG_FILE_SCAN_DIR);")/newrelic.ini

CMD ["php-fpm", "-F"]
