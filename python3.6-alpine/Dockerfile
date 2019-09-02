# Alpine 3.9 is the last version that uses Python 3.6
FROM alpine:3.9

# bash: for Invenio scripts
# openssl-dev: for cryptography pip lib
# libpq and postgresql-dev: for psycopg pip lib
RUN set -ex \
  && buildDeps=' \
    build-base \
    ca-certificates \
    libpq \
    linux-headers \
    openssl-dev \
    postgresql-dev \
    python3-dev \
  ' \
  && apk --no-cache --update add \
    bash \
    curl \
    git \
    cairo-dev \
    icu-libs \
    libffi-dev \
    libxml2-dev \
    libxslt-dev \
    nodejs \
    npm \
    $buildDeps \
  && pip3 install --no-cache-dir --upgrade pip \
  && pip3 install --no-cache-dir --upgrade pipenv setuptools wheel \
  && rm -rf /var/cache/apk/*

SHELL ["/bin/sh", "-c"]

# set the locale thanks to icu-libs
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Create working directory
ENV WORKING_DIR=/opt/invenio
ENV INVENIO_INSTANCE_PATH=${WORKING_DIR}/var/instance
RUN mkdir -p ${INVENIO_INSTANCE_PATH}

# Create files mountpoints
RUN mkdir ${INVENIO_INSTANCE_PATH}/data
RUN mkdir ${INVENIO_INSTANCE_PATH}/archive
RUN mkdir ${INVENIO_INSTANCE_PATH}/static

# copy everything inside /src
RUN mkdir -p ${WORKING_DIR}/src
WORKDIR ${WORKING_DIR}/src

# Set `npm` global under Invenio instance path
RUN mkdir ${INVENIO_INSTANCE_PATH}/.npm-global
ENV NPM_CONFIG_PREFIX=$INVENIO_INSTANCE_PATH/.npm-global
RUN mkdir npm_install && cd npm_install && \
    curl -SsL https://registry.npmjs.org/npm/-/npm-6.4.1.tgz | tar -xzf - && \
    cd package && \
    node bin/npm-cli.js rm npm -g && \
    node bin/npm-cli.js install -g $(node bin/npm-cli.js pack | tail -1) && \
    cd ../.. && rm -rf npm_install

RUN npm config set prefix '${INVENIO_INSTANCE_PATH}/.npm-global'
ENV PATH=${INVENIO_INSTANCE_PATH}/.npm-global/bin:$PATH

# Set folder permissions
ENV INVENIO_USER_ID=1000
RUN chgrp -R 0 ${WORKING_DIR} && \
    chmod -R g=u ${WORKING_DIR}
RUN adduser -D -u ${INVENIO_USER_ID} invenio root && \
    chown -R invenio:root ${WORKING_DIR}
