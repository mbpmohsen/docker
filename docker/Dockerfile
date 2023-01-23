ARG PHP_VERSION

FROM php:${PHP_VERSION}

ENV TZ=Asia\Tehran

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install system dependencies
RUN apt-get update && apt-get install -y \
      ca-certificates \
      curl \
      git \
      gosu \
      libcap2-bin \
      libcurl4-openssl-dev \
      libonig-dev \
      libpng-dev \
      libssl-dev \
      libxml2-dev \
      libzip-dev \
      pkg-config \
      python2 \
      sqlite3 \
      supervisor \
      unzip \
      zip

# Install & configure gnupg
RUN apt-get install -y gnupg2 \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E5267A6C \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C300EE8C

# Install PHP extensions (keep in sync them with composer.json required expentions [expect curl dom hash filter json libxml pdo])
RUN docker-php-ext-install pdo_mysql pcntl bcmath

# Install ext-gd
RUN apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libwebp-dev \
    libfreetype6-dev
RUN docker-php-ext-configure gd \
    --with-webp=/usr/include/ \
    --with-freetype=/usr/include/ \
    --with-jpeg=/usr/include/
RUN docker-php-ext-install gd

# Install ext-zip
RUN docker-php-ext-configure zip
RUN docker-php-ext-install zip

# Install ext-intl
RUN docker-php-ext-configure intl && docker-php-ext-install -j$(nproc) intl

# Install ext-redis
RUN pecl install redis && docker-php-ext-enable redis

# https://blog.denisbondar.com/post/phpstorm_docker_xdebug (RU)
RUN pecl install xdebug && docker-php-ext-enable xdebug

# https://www.hmazter.com/2019/04/speeding-up-php-docker-with-opcache/
RUN docker-php-ext-install opcache

# Install MySQL Client to use mysqldump and mysql CLI tools (needed to prepare DBs for parallel testing)
RUN apt-get install -y default-mysql-client

RUN apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


# PHP ini config
COPY ./docker/php/config/php.ini /usr/local/etc/php/conf.d/99-local.ini
COPY ./docker/php/config/xdebug.ini usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
COPY ./docker/php/config/opcache.ini /usr/local/etc/php/conf.d/99-opcache.ini
COPY ./docker/php/config/zz-www.conf /usr/local/etc/php-fpm.d/99-zz-www.conf

# Get latest Composer v2.2
COPY --from=composer:2.2 /usr/bin/composer /usr/bin/composer

ARG GROUP_ID
ARG USER_ID
ARG USER_NAME

# Inspired by https://github.com/hexops/dockerfile#do-not-use-a-uid-below-10000 and https://github.com/hexops/dockerfile/blob/main/Dockerfile
# Non-root user for security purposes.
#
# UIDs below 10,000 are a security risk, as a container breakout could result
# in the container being ran as a more privileged user on the host kernel with
# the same UID.
#
# Static GID/UID is also useful for chown’ing files outside the container where
# such a user does not exist.
RUN addgroup --gid $GROUP_ID --system $USER_NAME \
 && adduser --uid $USER_ID --system --ingroup $USER_NAME --home /home/$USER_NAME $USER_NAME

# Set working directory
WORKDIR /var/www

USER ${USER_NAME}
