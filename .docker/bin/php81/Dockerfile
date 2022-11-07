FROM php:8.1.11-apache-bullseye

# Avoid prompts when building
ENV DEBIAN_FRONTEND noninteractive

ARG USERNAME
ARG UID
ARG TIMEZONE
ARG SERVER_NAME
ARG NODE_VERSION

# Update
RUN apt-get -y update --fix-missing && \
    apt-get upgrade -y && \
    apt-get --no-install-recommends install -y apt-utils && \
    rm -rf /var/lib/apt/lists/*

# set Timezone
RUN ln -snf /usr/share/zoneinfo/$TIMEZONE /etc/localtime && echo $TIMEZONE > /etc/timezone

# Install useful tools and install important libaries
RUN apt-get -y update && \
    apt-get -y --no-install-recommends install nano wget \
            dialog \
            libsqlite3-dev \
            libsqlite3-0 && \
    apt-get -y --no-install-recommends install default-mysql-client \
            zlib1g-dev \
            libzip-dev \
            libicu-dev && \
    apt-get -y --no-install-recommends install --fix-missing apt-utils \
            build-essential \
            git \
            curl \
            libpq-dev \
            libonig-dev && \
    apt-get install -y iputils-ping && \
    apt-get -y --no-install-recommends install --fix-missing libcurl4 \
            libcurl4-openssl-dev \
            zip \
            unzip \
            openssl && \
    rm -rf /var/lib/apt/lists/*
    
# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
     
# Install Node.js
RUN mkdir /usr/local/nvm
ENV NVM_DIR /usr/local/nvm
RUN curl https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash \
    && . $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default

ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

RUN npm install -g yarn

# Install imagick
RUN apt-get update && \
    apt-get -y --no-install-recommends install --fix-missing libmagickwand-dev && \
    rm -rf /var/lib/apt/lists/*

# Workaround until imagick is available in pecl with php8 support
# Imagick Commit to install
# https://github.com/Imagick/imagick
ARG IMAGICK_COMMIT="661405abe21d12003207bc8eb0963fafc2c02ee4"

RUN cd /usr/local/src && \
    git clone https://github.com/Imagick/imagick && \
    cd imagick && \
    git checkout ${IMAGICK_COMMIT} && \
    phpize && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf imagick && \
    docker-php-ext-enable imagick

# Other PHP8 Extensions
RUN  docker-php-ext-install pdo_mysql
RUN  docker-php-ext-install mysqli
RUN  docker-php-ext-install bcmath
RUN  docker-php-ext-install curl
RUN  docker-php-ext-install zip
RUN  docker-php-ext-install -j$(nproc) intl
RUN  docker-php-ext-install mbstring
RUN  docker-php-ext-install gettext
RUN  docker-php-ext-install calendar
RUN  docker-php-ext-install exif
RUN  docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql
RUN  docker-php-ext-install pdo pdo_pgsql pgsql
RUN  docker-php-ext-install opcache

# Install Freetype 
RUN apt-get -y update && \
    apt-get --no-install-recommends install -y libfreetype6-dev \
            libjpeg62-turbo-dev \
            libpng-dev && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg && \
    docker-php-ext-install gd

# User and group
RUN useradd -G www-data,root -u $UID -d /home/$USERNAME $USERNAME
RUN mkdir -p /home/$USERNAME/.composer && \
    chown -R $USERNAME:$USERNAME /home/$USERNAME

RUN echo "ServerName " $SERVER_NAME >> /etc/apache2/apache2.conf
# Insure an SSL directory exists
RUN mkdir -p /etc/apache2/ssl

# Enable SSL support
RUN a2enmod ssl && a2enmod rewrite

# Enable apache modules
RUN a2enmod rewrite headers

# Cleanup
RUN rm -rf /usr/src/*

USER $USERNAME
