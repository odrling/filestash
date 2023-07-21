# STEP2: BUILD THE FRONTEND
FROM node:14-alpine AS builder_frontend
WORKDIR /home/
COPY . ./
RUN apk add make git && \
    npm install && \
    make build_frontend

# STEP3: BUILD THE BACKEND
FROM golang:1.19-buster AS builder_backend
WORKDIR /home/
COPY --from=builder_frontend /home/ ./
RUN apt-get update > /dev/null && \
    apt-get install -y libvips-dev curl make > /dev/null 2>&1 && \
    make build_init && \
    make build_backend && \
    mkdir -p ./dist/data/state/config/ && \
    cp config/config.json ./dist/data/state/config/config.json

# STEP4: Create the prod image from the build
FROM machines/filestash:latest
COPY --from=mwader/static-ffmpeg:latest /ffmpeg /ffprobe /usr/local/bin/
COPY --from=builder_backend --chown=filestash:filestash /home/dist/ /app/
