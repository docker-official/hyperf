# hyperf/hyperf:7.4

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

ENV TIMEZONE=${TIMEZONE:-"Asia/Shanghai"} \
    SW_VERSION=${SW_VERSION:-"v4.7.1"} \
    COMPOSER_VERSION=${COMPOSER_VERSION:-"2.1.8"}

# update
RUN set -ex \
    && apk update  \
    && apk add --no-cache libstdc++ openssl git bash  \
    && apk add --no-cache --virtual .build-deps autoconf dpkg-dev dpkg file g++ gcc libc-dev make php7-dev php7-pear \
                            pkgconf re2c pcre-dev pcre2-dev zlib-dev libtool automake libaio-dev openssl-dev curl-dev \
    # - php extension:swoole
    && cd /tmp  \
    && curl -SL "https://gitee.com/swoole/swoole/repository/archive/${SW_VERSION}.tar.gz" -o swoole.tar.gz  \
    && ls -alh  \
    && cd /tmp  \
    && mkdir -p swoole  \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1  \
    && ln -s /usr/bin/phpize7 /usr/local/bin/phpize  \
    && ln -s /usr/bin/php-config7 /usr/local/bin/php-config  \
    && (  \
        cd swoole \
        && phpize \
        && ./configure --enable-openssl --enable-http2 --enable-swoole-curl --enable-swoole-json \
        && make -s -j$(nproc) && make install  \
    )  \
    && echo "memory_limit=1G" > /etc/php7/conf.d/00_default.ini  \
    && echo "opcache.enable_cli = 'On'" >> /etc/php7/conf.d/00_opcache.ini  \
    && echo "extension=swoole.so" > /etc/php7/conf.d/50_swoole.ini  \
    && echo "swoole.use_shortname = 'Off'" >> /etc/php7/conf.d/50_swoole.ini  \
    # - install composer
    && wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar  \
    && chmod u+x /usr/local/bin/composer  \
    # - show php version and extensions
    && php -v  \
    && php -m  \
    && php --ri swoole  \
    && composer  \
    # - config timezone
    && ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone \
    # - clear works
    && apk del .build-deps  \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/local/bin/php*  \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"
