# define an alias for the specific php version used in this file.
FROM php:8.3-fpm AS base

# Python build stage
FROM base AS php
LABEL name=bedrock
LABEL intermediate=true
ARG BUILD_ENVIRONMENT=local
ARG APP_HOME=/app

# Install essential packages
RUN apt-get update \
  && apt-get install -y \
    build-essential \
    curl \
    git \
    gnupg \
    less \
    nano \
    vim \
    unzip \
    zip \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get clean

# Install php extensions and related packages
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions && sync \
  && install-php-extensions \
    @composer \
    exif \
    gd \ 
    memcached \
    mysqli \
    pcntl \
    pdo_mysql \
    zip \
  && apt-get update \
  && apt-get install -y \
    gifsicle \
    jpegoptim \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libmemcached-dev \
    locales \
    lua-zlib-dev \
    optipng \
    pngquant \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get clean

# Install nginx & supervisor
RUN curl -sL https://deb.nodesource.com/setup_20.x | bash \
  && apt-get update \
  && apt-get install -y \
    nginx \
    nodejs \
    supervisor \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get clean \
  && npm install -g yarn

# Start install
RUN mkdir /var/www/pandacode
COPY ./.envs/.local/supervisord.conf /etc/supervisord.conf
COPY ./.envs/.local/wp.conf /var/www/pandacode/.env
COPY ./config /var/www/pandacode/config
COPY ./web /var/www/pandacode/web
COPY ./composer.json /var/www/pandacode/composer.json
RUN cd /var/www/pandacode && composer install
RUN cd /var/www/pandacode/web/app/themes/pandacode && composer install && yarn && yarn build

COPY ./.envs/.local/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./.envs/.local/nginx/sites-enabled /etc/nginx/conf.d
COPY ./.envs/.local/nginx/sites-enabled /etc/nginx/sites-enabled

RUN chmod -R 777 /var/www/pandacode/web/app

COPY ./compose/local/php/start /var/www/pandacode/start.sh
RUN chmod +x /var/www/pandacode/start.sh

# default entry point.
ENTRYPOINT ["sh", "/var/www/pandacode/start.sh"]
# Install lib nessessary
# RUN apt-get install -y \
#     git \
#     curl \
#     wget \
#     build-essential \
#     zlib1g-dev \
#     liblzma-dev \
#     libfreetype6-dev \
#     libssl-dev \
#     libpng-dev \
#     libjpeg-dev \
#     icc-profiles-free \
#     libxrender1 \
#     libxext6 \
#     libx11-6 \
#     fontconfig \
#     xfonts-base \
#     xfonts-75dpi \
#     default-libmysqlclient-dev \
#     gettext \
#     ffmpeg \
#     vlc

# Requirements are installed here to ensure they will be cached.
# COPY ./config ${APP_HOME}/config
# COPY ./web ${APP_HOME}/web
# COPY ./composer.json ${APP_HOME}/composer.json

# Create Python Dependency and Sub-Dependency Wheels.
# RUN pip wheel --wheel-dir /usr/src/app/wheels  \
#   -r ${BUILD_ENVIRONMENT}.txt


# # Python 'run' stage
# FROM python AS python-run-stage

# ARG BUILD_ENVIRONMENT=local
# ARG APP_HOME=/app

# ENV PYTHONUNBUFFERED=1
# ENV PYTHONDONTWRITEBYTECODE=1
# ENV BUILD_ENV=${BUILD_ENVIRONMENT}

# WORKDIR ${APP_HOME}


# # devcontainer dependencies and utils
# RUN apt-get update && apt-get install --no-install-recommends -y default-libmysqlclient-dev ffmpeg \
#   sudo git bash-completion nano ssh

# # Create devcontainer user and add it to sudoers
# RUN groupadd --gid 1000 dev-user \
#   && useradd --uid 1000 --gid dev-user --shell /bin/bash --create-home dev-user \
#   && echo dev-user ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/dev-user \
#   && chmod 0440 /etc/sudoers.d/dev-user


# # Install required system dependencies
# RUN apt-get update && apt-get install --no-install-recommends -y \
#   # psycopg dependencies
#   libpq-dev \
#   # Translations dependencies
#   gettext \
#   ffmpeg \
#   # cleaning up unused files
#   && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
#   && rm -rf /var/lib/apt/lists/*

# # All absolute dir copies ignore workdir instruction. All relative dir copies are wrt to the workdir instruction
# # copy python dependency wheels from python-build-stage
# COPY --from=python-build-stage /usr/src/app/wheels  /wheels/

# # use wheels to install python dependencies
# RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
#   && rm -rf /wheels/

# COPY ./compose/production/django/entrypoint /entrypoint
# RUN sed -i 's/\r$//g' /entrypoint
# RUN chmod +x /entrypoint

# COPY ./compose/local/django/start /start
# RUN sed -i 's/\r$//g' /start
# RUN chmod +x /start


# COPY ./compose/local/django/celery/worker/start /start-celeryworker
# RUN sed -i 's/\r$//g' /start-celeryworker
# RUN chmod +x /start-celeryworker

# COPY ./compose/local/django/celery/beat/start /start-celerybeat
# RUN sed -i 's/\r$//g' /start-celerybeat
# RUN chmod +x /start-celerybeat

# COPY ./compose/local/django/celery/flower/start /start-flower
# RUN sed -i 's/\r$//g' /start-flower
# RUN chmod +x /start-flower


# # copy application code to WORKDIR
# COPY . ${APP_HOME}

# ENTRYPOINT ["/entrypoint"]
