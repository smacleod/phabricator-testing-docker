FROM php:5.6-fpm-alpine

# Git commit SHAs for the build artifacts we want to grab.
# From https://github.com/phacility/phabricator/commits/stable
# Promote 2018 Week 12 plus the following extra commits:
# (stable) Drop uniqueness constraint on PushEvent request ID
ENV PHABRICATOR_GIT_SHA 5e710959bed93b75f239821b739e55d0b8a63122
# From https://github.com/phacility/arcanist/commits/stable
# Promote 2018 Week 10
ENV ARCANIST_GIT_SHA 7b6a9a93f4a4991b5173a2e9a1b2d8e4f630405f

# From https://github.com/phacility/libphutil/commits/stable
# Promote 2018 Week 11
ENV LIBPHUTIL_GIT_SHA 47cfa511ca6782df13c459fdd212606f091a44ec
# Should match the phabricator 'repository.default-local-path' setting.
ENV REPOSITORY_LOCAL_PATH /repo
# Explicitly set TMPDIR
ENV TMPDIR /tmp

# Runtime dependencies
RUN apk --no-cache --update add \
    curl \
    freetype \
    g++ \
    git \
    libjpeg-turbo \
    libmcrypt \
    libpng \
    make \
    mariadb-client \
    mariadb-client-libs \
    ncurses \
    py-pygments

RUN \
  apk add python-dev py-pip && \
  pip install Mercurial

# Build PHP extensions
RUN \
  apk --no-cache add \
    --virtual build-dependencies \
    $PHPIZE_DEPS \
    curl-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    mariadb-dev \
  && docker-php-ext-configure gd \
    --with-freetype-dir=/usr/include \
    --with-jpeg-dir=/usr/include \
    --with-png-dir=/usr/include \
  && docker-php-ext-install -j "$(nproc)" \
    curl \
    gd \
    iconv \
    mbstring \
    mcrypt \
    mysqli \
    opcache \
    pcntl \
  && pecl install https://s3.amazonaws.com/net-mozaws-dev-mozphab-pecl-mirror/apcu-4.0.11.tgz \
  && docker-php-ext-enable apcu \
  && apk del build-dependencies

RUN wget -O /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64 && if test -f /usr/local/bin/dumb-init; then chmod 755 /usr/local/bin/dumb-init; fi

# Install opcache recommended settings from
# https://secure.php.net/manual/en/opcache.installation.php
RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
        echo 'opcache.validate_timestamps=0'; \
    } | tee /usr/local/etc/php/conf.d/opcache.ini

# The container does not log errors by default, so turn them on
RUN { \
        echo 'php_admin_flag[log_errors] = on'; \
        echo 'php_flag[display_errors] = off'; \
    } | tee /usr/local/etc/php-fpm.d/zz-log.conf

# Phabricator recommended settings (skipping these will result in setup warnings
# in the application).
RUN { \
        echo 'always_populate_raw_post_data=-1'; \
        echo 'post_max_size="32M"'; \
    } | tee /usr/local/etc/php/conf.d/phabricator.ini

COPY ./app /app
WORKDIR /app

# Install Phabricator code
RUN \
  curl -fsSL https://github.com/phacility/phabricator/archive/${PHABRICATOR_GIT_SHA}.tar.gz -o phabricator.tar.gz \
  && curl -fsSL https://github.com/phacility/arcanist/archive/${ARCANIST_GIT_SHA}.tar.gz -o arcanist.tar.gz \
  && curl -fsSL https://github.com/phacility/libphutil/archive/${LIBPHUTIL_GIT_SHA}.tar.gz -o libphutil.tar.gz \
  && tar xzf phabricator.tar.gz \
  && tar xzf arcanist.tar.gz \
  && tar xzf libphutil.tar.gz \
  && mv phabricator-${PHABRICATOR_GIT_SHA} phabricator \
  && mv arcanist-${ARCANIST_GIT_SHA} arcanist \
  && mv libphutil-${LIBPHUTIL_GIT_SHA} libphutil \
  && rm phabricator.tar.gz arcanist.tar.gz libphutil.tar.gz \
  && ./libphutil/scripts/build_xhpast.php

RUN \
  chmod +x /app/entrypoint.sh /app/wait-for-mysql.php \
  && mkdir -p $REPOSITORY_LOCAL_PATH \

VOLUME ["$REPOSITORY_LOCAL_PATH"]
EXPOSE 9000
ENTRYPOINT ["/usr/local/bin/dumb-init", "--"]
CMD ["/app/entrypoint.sh", "start"]
