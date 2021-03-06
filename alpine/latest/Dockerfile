###############################################################
#
# Copyright (C) 2019 James Fuller <jim.fuller@webcomposite.com>
#
# SPDX-License-Identifier: MIT
#
# pinning to Alpine 3.10.3 as base image
###############################################################
FROM registry.hub.docker.com/library/alpine:3.11.3 AS builder

###############################################################
# set build args
###############################################################
ARG CURL_RELEASE_TAG=latest
ARG CURL_GIT_REPO=https://github.com/curl/curl.git
ARG CURL_CONFIGURE_OPTION
ARG LABEL_VERSION=1.0.0
ARG LABEL_NAME=curl
ARG LABEL_DESC=curl

###############################################################
# build curl
###############################################################
# install deps and use latest curl release source
RUN \
    apk add \
       libssh2-dev      \
       libssh2          \
       libssh2-static    \
       autoconf     \
       automake     \
       build-base   \
       groff        \
       curl-dev     \
       openssl      \
       libtool

RUN mkdir /src
COPY "curl"  "/src/curl"
WORKDIR "/src/curl"

RUN ./buildconf && \
    autoreconf -vif && \
    ./configure ${CURL_CONFIGURE_OPTION} &&\
    make -j$(nproc) &&\
    make DESTDIR="/alpine/" install  -j$(nproc)

###############################################################
# pinning to Alpine 3.11.3
###############################################################
FROM registry.hub.docker.com/library/alpine:3.11.3

ARG CURL_RELEASE_TAG=latest
ARG CURL_RELEASE_VERSION
ARG CURL_GIT_REPO=https://github.com/curl/curl.git

ENV CURL_VERSION ${CURL_RELEASE_VERSION}
ENV CURL_RELEASE_TAG ${CURL_RELEASE_TAG}
ENV CURL_GIT_REPO ${CURL_GIT_REPO}

###############################################################
# define docker labels
###############################################################
LABEL Maintainer="James Fuller <jim.fuller@webcomposite.com>"
LABEL Name="curl"
LABEL Version="${LABEL_VERSION}"
LABEL docker.cmd="docker run -it curl/curl:7.69.0 http://curl.haxx.se"

###############################################################
# dependencies
###############################################################
RUN apk add --no-cache nghttp2 libssh2 ca-certificates curl

###############################################################
# add non privileged curl user
###############################################################
RUN addgroup -S curl_group && adduser -S curl_user -G curl_group

###############################################################
# install curl ca bundle
###############################################################
RUN curl https://curl.haxx.se/ca/cacert.pem  -o cacert.pem
ENV CURL_CA_BUNDLE="/cacert.pem"
RUN apk del --no-cache curl ca-certificates

###############################################################
# install curl built from builder
###############################################################
COPY --from=builder "/alpine/usr/local/lib/libcurl.so" "/usr/lib/libcurl.so"
COPY --from=builder "/alpine/usr/local/lib/libcurl.so" "/usr/lib/libcurl.so"
COPY --from=builder "/alpine/usr/local/lib/libcurl.so.4.6.0" "/usr/lib/libcurl.so.4"
COPY --from=builder "/alpine/usr/local/bin/curl" "/usr/bin/curl"

###############################################################
# set user
###############################################################
USER curl_user

###############################################################
# set entrypoint
###############################################################
COPY "entrypoint.sh" "/entrypoint.sh"
ENTRYPOINT ["/entrypoint.sh"]
CMD ["curl"]
