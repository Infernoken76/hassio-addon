# Stap 1: Frontend
FROM node:20-alpine AS frontend-builder
RUN apk add --no-cache git
WORKDIR /app
ARG CACHE_VERSION=v1.2
RUN git clone --depth 1 https://github.com/Infernoken76/evcc.git .
WORKDIR /app/web
RUN npm ci
RUN npm run build

# Stap 2: Backend
FROM golang:1.26-alpine AS backend-builder
RUN apk add --no-cache git gcc musl-dev linux-headers
WORKDIR /go/src/github.com/evcc-io/evcc
COPY --from=frontend-builder /app/go.mod /app/go.sum ./
RUN go mod download
COPY --from=frontend-builder /app .
RUN go build -ldflags="-s -w" -tags release -o evcc .

# Stap 3: Het uiteindelijke compacte Home Assistant Add-on image
FROM alpine:latest
RUN apk add --no-cache tzdata ca-certificates

# We werken in de map waar de i18n bestanden staan
WORKDIR /opt/evcc

# Kopieer de binary en de vertaalbestanden
COPY --from=backend-builder /go/src/github.com/evcc-io/evcc/evcc /usr/local/bin/evcc
COPY --from=backend-builder /go/src/github.com/evcc-io/evcc/i18n /opt/evcc/i18n

# --- DIT IS DE AANPASSING ---
# We dwingen de container om jouw evcc.yaml in de map /data te lezen
# (Home Assistant koppelt jouw addon_configs map automatisch aan /data)
# We wijzen de container naar het configuratiebestand in de nieuwe map
# Gebruik ENTRYPOINT voor het commando en CMD voor de extra argumenten
ENTRYPOINT ["evcc"]
CMD ["--config", "/share/evcc/evcc.yaml"]
