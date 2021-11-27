# hyperf/hyperf:7.3

ARG ALPINE_VERSION=3.11

FROM hyperf/hyperf:7.4-alpine-v${ALPINE_VERSION}-base

LABEL maintainer="JohnHe China <564079964@qq.com>"

##
# ---------- env settings ----------
##
# --build-arg TIMEZONE=Asia/Shanghai
ARG TIMEZONE

ARG SW_VERSION
ARG COMPOSER_VERSION

ENV TIMEZONE_ENV=${TIMEZONE:-"Asia/Shanghai"} \
    SW_VERSION=${SW_VERSION:-"v4.5.7"} \
    COMPOSER_VERSION=${COMPOSER_VERSION:-"2.0.2"}

COPY ./swoole.tar.gz /tmp

# update
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    set -ex \
    && apk update \
    # - for swoole extension libaio linux-headers
    && apk add --no-cache libstdc++ openssl git bash \
    && apk add --no-cache --virtual .build-deps autoconf dpkg-dev dpkg file g++ gcc libc-dev make php7-dev php7-pear \
                            pkgconf re2c pcre-dev pcre2-dev zlib-dev libtool automake libaio-dev openssl-dev curl-dev \
    # - php extension:swoole
    && cd /tmp \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && ln -s /usr/bin/phpize7 /usr/local/bin/phpize \
    && ln -s /usr/bin/php-config7 /usr/local/bin/php-config \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-openssl --enable-http2 --enable-swoole-curl --enable-swoole-json \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /etc/php7/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /etc/php7/conf.d/00_opcache.ini \
    && echo "extension=swoole.so" > /etc/php7/conf.d/50_swoole.ini \
    && echo "swoole.use_shortname = 'Off'" >> /etc/php7/conf.d/50_swoole.ini \
    # - install composer
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    # - show php version and extensions
    && php -v \
    && php -m \
    && php --ri swoole \
    && composer \
    # - config timezone
    && ln -sf /usr/share/zoneinfo/${TIMEZONE_ENV} /etc/localtime \
    && echo "${TIMEZONE_ENV}" > /etc/timezone \
    # - clear works
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/local/bin/php* \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"
