FROM node:13-alpine as frontend

COPY . /src
RUN rm -r /src/server
WORKDIR /src

RUN apk add make git
RUN npm i
RUN make build_frontend

FROM golang:1.19-buster as backend

ENV DEBIAN_FRONTEND noninteractive
ENV CGO_LDFLAGS_ALLOW -fopenmp
ENV GO111MODULE on

COPY . /src
RUN rm -r /src/client
WORKDIR /src

RUN apt-get update && apt install -y libglib2.0-dev curl make

RUN make build_init

# some fucking libs
ADD https://hikari.butaishoujo.moe/b/c158f709/libresize_linux_amd64.a /src/server/plugin/plg_image_light/deps/
ADD https://hikari.butaishoujo.moe/b/e3d99ec0/libtranscode_linux_amd64.a /src/server/plugin/plg_image_light/deps/

COPY --from=frontend /src/server/ctrl/static/www /src/server/ctrl/static/www
RUN make build_backend
RUN timeout 1 ./dist/filestash || true

FROM debian:stable-slim

RUN apt-get update && \
    apt-get install -y libglib2.0-0 curl gnupg && \
    apt-get install -y tor emacs zip poppler-utils

RUN mkdir -p /filestash/data/state/config
COPY --from=backend /src/dist/filestash /filestash
COPY config /filestash/data/state/config

ADD https://hikari.butaishoujo.moe/b/6833cd58/ffmpeg /usr/bin/ffmpeg
ADD https://hikari.butaishoujo.moe/b/05bf87a9/ffprobe /usr/bin/ffprobe
RUN chmod +x /usr/bin/ffmpeg /usr/bin/ffprobe

RUN useradd filestash && \
    chown -R filestash:filestash /filestash/

#USER filestash

RUN chmod -R o+rwx /filestash/data

EXPOSE 8334

ENTRYPOINT /filestash/filestash
