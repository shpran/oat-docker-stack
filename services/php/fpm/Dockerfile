FROM php:7.4.28-fpm

ARG PROJECT_NAME

ARG GITHUB_USER
ARG GITHUB_TOKEN
ARG GITHUB_EMAIL
ARG GITHUB_NAME

ARG NPM_TOKEN

RUN apt update \
    && apt install -y \
        debian-archive-keyring \
        libonig-dev \
        libpng-dev \
        libjpeg-dev \
        libpq-dev \
        libzip-dev \
        libzstd-dev \
        libcurl4-gnutls-dev \
        libsqlite3-dev \
        libnss3-tools \
        libxrender1 \
        zip \
        unzip \
        sudo \
        nano \
        git \
        wget \
        curl \
        sqlite3 \
        xfonts-75dpi \
        xfonts-base \
        fontconfig \
    && rm -rf /var/lib/apt/lists/*

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && composer config -g github-protocols https \
    && composer config -g process-timeout 0

RUN wget 'https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.stretch_amd64.deb' \
    && dpkg -i wkhtmltox_0.12.5-1.stretch_amd64.deb \
    && rm -f wkhtmltox_0.12.5-1.stretch_amd64.deb \
    && ln -s /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf \
    && apt-get -f install

RUN curl -sL https://deb.nodesource.com/setup_14.x | bash - \
    && apt-get install -yq nodejs build-essential \
    && npm install -g npm
RUN wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash \
    && export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")" \
    && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

RUN wget https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash \
    && mv git-completion.bash /usr/local/share/.git-completion.bash

RUN git config --global user.email "${GITHUB_EMAIL}" \
    && git config --global user.name "${GITHUB_NAME}" \
    && git config --global credential.helper store
RUN touch ~/.git-credentials \
    && echo https://$GITHUB_USER:$GITHUB_TOKEN@github.com > ~/.git-credentials

ADD ./bashrc_config.txt /usr/local/share/bashrc_config.txt
RUN cat >> ~/.bashrc /usr/local/share/bashrc_config.txt \
    && /bin/bash -c 'source ~/.bashrc'

RUN docker-php-ext-configure gd --with-jpeg=/usr \
    && docker-php-ext-configure pdo_mysql --with-pdo-mysql=mysqlnd \
    && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql

RUN yes | pecl install \
    igbinary \
    redis \
    xdebug

RUN docker-php-ext-install pdo \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install pgsql \
    && docker-php-ext-install pdo_pgsql \
    && docker-php-ext-install gd \
    && docker-php-ext-install opcache \
    && docker-php-ext-install zip \
    && docker-php-ext-install curl \
    && docker-php-ext-install calendar \
    && docker-php-ext-install pcntl \
    && docker-php-ext-enable igbinary \
    && docker-php-ext-enable xdebug \
    && docker-php-ext-enable redis

RUN echo //registry.npmjs.org/:_authToken=$NPM_TOKEN > /root/.npmrc

RUN npm i -g @oat-sa/tao-extension-release
RUN { \
    echo '{'; \
    echo '  "token": "'$GITHUB_TOKEN'"'; \
    echo '}'; \
} >> ~/.tao-extension-release

RUN npm i -g @oat-sa/tao-release-notes
RUN { \
    echo '{'; \
    echo '  "token": "'$GITHUB_TOKEN'"'; \
    echo '}'; \
} >> ~/.tao-release-notes

RUN mkdir /var/www/tools
RUN mkdir /var/www/tools/php-cs-fixer \
    && composer require --working-dir=/var/www/tools/php-cs-fixer friendsofphp/php-cs-fixer

RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && architecture=$(uname -m) \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/$architecture/$version \
    && mkdir -p /tmp/blackfire \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
    && mv /tmp/blackfire/blackfire-*.so $(php -r "echo ini_get ('extension_dir');")/blackfire.so \
    && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8307\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
    && rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz

RUN docker-php-source delete \
    && apt-get autoremove --purge -y && apt-get autoclean -y && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* /var/tmp/*

RUN mkdir /var/www/$PROJECT_NAME
WORKDIR /var/www/$PROJECT_NAME
