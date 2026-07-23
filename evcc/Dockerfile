# Stap 1: Frontend
FROM node:20-alpine AS frontend-builder
RUN apk add --no-cache git
WORKDIR /app
ARG CACHE_VERSION=v1.3
RUN git clone --depth 1 https://github.com/Infernoken76/evcc.git .
WORKDIR /app/web
RUN npm ci
RUN npm run build

# Stap 2: Backend
FROM golang:1.23-alpine AS backend-builder
RUN apk add --no-cache git gcc musl-dev linux-headers wget && \
    wget -O /usr/local/share/ca-certificates/isrg-root-yr.crt \
        https://letsencrypt.org/certs/gen-y/root-yr.pem && \
    update-ca-certificates
WORKDIR /go/src/github.com/evcc-io/evcc
COPY --from=frontend-builder /app/go.mod /app/go.sum ./
RUN go mod download
COPY --from=frontend-builder /app .
RUN go build -ldflags="-s -w" -tags release -o evcc .

# Stap 3: Finale image
FROM alpine:3.19
RUN apk add --no-cache tzdata ca-certificates wget && \
    wget -O /usr/local/share/ca-certificates/isrg-root-yr.crt \
        https://letsencrypt.org/certs/gen-y/root-yr.pem && \
    update-ca-certificates
WORKDIR /opt/evcc
COPY --from=backend-builder /go/src/github.com/evcc-io/evcc/evcc /usr/local/bin/evcc
COPY --from=backend-builder /go/src/github.com/evcc-io/evcc/i18n /opt/evcc/i18n
ENTRYPOINT ["evcc"]
CMD ["--config", "/config/evcc.yaml", "--database", "/data/evcc.db"]
