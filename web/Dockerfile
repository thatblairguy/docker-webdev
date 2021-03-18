FROM ubuntu:focal

# Update the package manager...
# Install add-apt-repository
# Add the PHP 7.3 repository
# ...then install packages.
RUN     DEBIAN_FRONTEND=noninteractive  apt-get update -y;                                \
        DEBIAN_FRONTEND=noninteractive  apt-get install -yq software-properties-common;  \
        DEBIAN_FRONTEND=noninteractive  add-apt-repository ppa:ondrej/php;               \
        DEBIAN_FRONTEND=noninteractive  add-apt-repository ppa:ondrej/apache2;           \
        DEBIAN_FRONTEND=noninteractive  apt-get install -yq                              \
        apache2                 \
        build-essential         \
        php7.4                  \
        libapache2-mod-php7.4   \
        php7.4-bz2              \
        php7.4-cli              \
        php7.4-common           \
        php7.4-curl             \
        php7.4-fpm              \
        php7.4-gd               \
        php7.4-json             \
        php7.4-mbstring         \
        php7.4-memcached        \
        php7.4-mysql            \
        php7.4-oauth            \
        php7.4-opcache          \
        php7.4-readline         \
        php7.4-sqlite3          \
        php7.4-soap             \
        php7.4-xdebug           \
        php7.4-xml              \
        mariadb-client          \
        curl                    \
        git                     \
        imagemagick             \
        vim                     \
        zip

# Copy up the available sites config. The startup script
# manipulates this file based on env vars.
COPY ./build/000-default.conf /etc/apache2/sites-available/000-default.conf

# ## These are the PHP modules that are available to be
# ## loaded by the CLI & apache. This should be inline with
# ## the modules installed above.
# COPY ./build/php-conf /etc/php/7.3/mods-available

# Enable mod_rewrite 'cause we needs it.
RUN a2enmod rewrite

# Forward logs to docker log collector.
RUN ln -sf /dev/stdout /var/log/apache2/access.log &&               \
    ln -sf /dev/stderr /var/log/apache2/error.log &&                \
    ln -sf /dev/stdout /var/log/apache2/000_default-access_log &&   \
    ln -sf /dev/stderr /var/log/apache2/000_default-error_log

# Copy our custom entrypoint and make it executable.
COPY ./build/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

# Install Node
# nvm environment variables
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 14.16.0

# install nvm
# https://github.com/creationix/nvm#install-script
RUN curl --silent -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.2/install.sh | bash

# install node and npm
RUN . $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default

# add node and npm to path so the commands are available
ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

# update npm
RUN npm install -g npm@7.6

## Turn off xdebug as it uses resources that could be best used
## elsewhere. Once the container is up you can use
## phpenmod -s <SAPI> xdebug
## where SAPI is probably, most likely, apache2
RUN phpdismod -s cli xdebug
RUN phpdismod -s apache2 xdebug

# # Expose the default Apache ports.
EXPOSE 80
EXPOSE 443

# Replace the standard entrypoint /bin/sh with our script.
ENTRYPOINT ["docker-entrypoint.sh"]

# If no command is passed to the container, start Apache by default.
CMD ["apachectl", "-D", "FOREGROUND"]

# Default to /var/www so we don't need to do it by hand.
WORKDIR /var/www
