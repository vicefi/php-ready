FROM php:8.1-fpm-alpine

ARG UID=1000
ARG GID=1000

RUN set -eux; \
    apk add --update --no-cache shadow; \
	groupmod -o -g ${GID} www-data; \
	usermod -o -g ${GID} -u ${UID} www-data; \
    apk del shadow

# Set timezone
#RUN apk add --update --no-cache tzdata
#RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
#&& echo ${TIMEZONE} > /etc/timezone \
#&& printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini \
#&& "date"

# Install php modules
RUN set -eux; \
    apk add --update --no-cache --virtual .dependies autoconf dpkg-dev dpkg file g++ gcc libc-dev make postgresql-dev pkgconf re2c bash; \
    apk add --no-cache libpq; \
    # pdo-mysql
    docker-php-ext-install pdo pdo_mysql; \
    docker-php-ext-enable pdo_mysql; \
    # pgsql
    docker-php-ext-install pgsql pdo_pgsql; \
    docker-php-ext-enable pdo_pgsql; \
    # bcmath
    docker-php-ext-install bcmath; \
    docker-php-ext-enable bcmath; \
    # pcntl
    docker-php-ext-install pcntl; \
    docker-php-ext-enable pcntl; \
    # gd
    apk add --update --no-cache zlib zlib-dev freetype freetype-dev libpng libpng-dev libjpeg-turbo libjpeg-turbo-dev; \
    docker-php-ext-configure gd \
       --with-freetype \
       --with-jpeg; \
    docker-php-ext-install gd; \
    docker-php-ext-enable gd; \
    # zip
    apk add --update --no-cache libzip-dev; \
    docker-php-ext-install zip; \
    docker-php-ext-enable zip; \
    # redis
    pecl install -o -f redis; \
    docker-php-ext-enable redis; \
    apk del --no-network .dependies; \
    rm -rf /var/cache/apk/*; \
    # composer
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=2.0.14

ARG XDEBUG_VERSION=3.1.3
RUN set -eux; \
    apk add --no-cache --virtual .build-deps ${PHPIZE_DEPS}; \
    pecl install xdebug-${XDEBUG_VERSION}; \
    docker-php-ext-enable xdebug; \
    apk del .build-deps

COPY bin/. /usr/local/bin/
COPY etc/docker-php-config.ini /usr/local/etc/php/conf.d/
COPY etc/php-fpm.conf /usr/local/etc/php-fpm.d/

## Add the wait script to the image
ARG WAIT_VERSION=2.8.0
ADD https://github.com/ufoscout/docker-compose-wait/releases/download/${WAIT_VERSION}/wait /usr/local/bin/wait-for-it
RUN chmod +x /usr/local/bin/wait-for-it

ARG APP_DIR=/var/www/html
ARG APP_ENV=dev

ENV APP_DIR ${APP_DIR}
ENV APP_ENV ${APP_ENV}

RUN set -eux; \
    mkdir -p ${APP_DIR}; \
    chown -R www-data:www-data ${APP_DIR};

WORKDIR ${APP_DIR}

VOLUME [ "${APP_DIR}" ]

USER www-data

CMD ["php-fpm"]

EXPOSE 9000
