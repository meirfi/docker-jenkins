FROM jenkins/jnlp-slave:3.23-1-alpine
ARG GOSU_VERSION=1.10
ARG DOCKER_VERSION=17.09.0-ce
ARG TINY_VERSION=0.16.1
ENV ALPINE_VERSION=3.16
USER root


ENV PACKAGES="\
      curl \
      wget \
      ca-certificates \
      shadow \
      py-pip \
"

# Replacing defult repos

#RUN \
#   echo "http://dl-cdn.alpinelinux.org/alpine/v$ALPINE_VERSION/testing" > /etc/apk/repositories \
#  && echo "http://dl-cdn.alpinelinux.org/alpine/v$ALPINE_VERSION/community" >> /etc/apk/repositories \
#  && echo "http://dl-cdn.alpinelinux.org/alpine/v$ALPINE_VERSION/main" >> /etc/apk/repositories   


RUN ALPINE_GLIBC_BASE_URL="https://github.com/sgerrand/alpine-pkg-glibc/releases/download" && \
    ALPINE_GLIBC_PACKAGE_VERSION="2.26-r0" && \
    ALPINE_GLIBC_BASE_PACKAGE_FILENAME="glibc-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_BIN_PACKAGE_FILENAME="glibc-bin-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_I18N_PACKAGE_FILENAME="glibc-i18n-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    apk update && \
    apk add --virtual=.build-dependencies wget ca-certificates shadow && \
#    apk add --no-cache $PACKAGES && \
#    apk add --virtual=.build-dependencies && \
    wget -q \
        "https://raw.githubusercontent.com/andyshinn/alpine-pkg-glibc/master/sgerrand.rsa.pub" \
        -O "/etc/apk/keys/sgerrand.rsa.pub" && \
    wget -q \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    apk add --no-cache \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    \
    rm "/etc/apk/keys/sgerrand.rsa.pub" && \
    /usr/glibc-compat/bin/localedef -i en_US -f UTF-8 en_US.UTF-8 && \
    echo "export LANG=en_US.UTF-8" > /etc/profile.d/locale.sh && \
    \
    apk del glibc-i18n && \
    \
    apk del .build-dependencies && \
    rm \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME"
ENV LANG=en_US.UTF-8

RUN \
    apk add curl curl-dev bash shadow py-pip \
    && curl -SsLo /usr/bin/gosu https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-amd64 \
    && chmod +x /usr/bin/gosu \
    && curl -SsLo /usr/bin/tiny https://github.com/krallin/tini/releases/download/v${TINY_VERSION}/tini-amd64 \
    && chmod +x /usr/bin/tiny

#RUN sudo -H pip install --upgrade pip
RUN python -m pip install -U  pip    
RUN \
    curl -SsLo /tmp/docker.tar.gz https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_VERSION}.tgz \
#    && pip install --upgrade pip \
    && tar -xzf /tmp/docker.tar.gz -C /tmp \
    && mv /tmp/docker/docker /usr/bin/docker \
    && rm -rf /tmp/docker /tmp/docker.tar.gz \
    && pip install docker-compose

COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT [ "tiny", "--", "/entrypoint.sh" ]
