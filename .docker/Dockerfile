# syntax=docker/dockerfile:1.4

FROM node:latest AS node-builder

COPY ./seanime-web /tmp/build
WORKDIR /tmp/build
RUN npm ci
RUN npm run build

FROM golang:latest AS go-builder

COPY . /tmp/build
COPY --from=node-builder /tmp/build/out /tmp/build/web
WORKDIR /tmp/build

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o seanime -trimpath -ldflags="-s -w"

FROM debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        gnupg \
        ca-certificates && \
    curl -fsSL https://repo.jellyfin.org/ubuntu/jellyfin_team.gpg.key | gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg && \
    echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/debian bookworm main' > /etc/apt/sources.list.d/jellyfin.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        intel-media-va-driver \
        intel-opencl-icd \
        jellyfin-ffmpeg7 && \
    rm -rf /tmp/* /var/lib/apt/lists/* /var/tmp/*
    
ENV PATH="/usr/lib/jellyfin-ffmpeg/:$PATH"

COPY --from=go-builder /tmp/build/seanime /app/

WORKDIR /app

EXPOSE 43211

CMD ["/app/seanime"]
