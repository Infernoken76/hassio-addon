# Stap 1: Bouw de web-interface (Frontend)
FROM node:20-alpine AS frontend-builder
RUN apk add --no-cache git
WORKDIR /app
RUN git clone --depth 1 https://github.com/Infernoken76/evcc.git .
WORKDIR /app/web
RUN npm ci
RUN npm run build

# Stap 2: Bouw de Go-applicatie (Backend) en voeg alles samen
FROM golang:1.26-alpine AS backend-builder
RUN apk add --no-cache git gcc musl-dev
WORKDIR /go/src/github.com/evcc-io/evcc

# --- WIJZIGING HIER: Eerst modules ophalen om de 3-minuten timeout te omzeilen ---
# Kopieer specifiek alleen de module-bestanden uit de frontend-builder
COPY --from=frontend-builder /app/go.mod /go/src/github.com/evcc-io/evcc/
COPY --from=frontend-builder /app/go.sum /go/src/github.com/evcc-io/evcc/

# Download alle Go-dependencies (dit krijgt nu een eigen, schone 3-minuten timer)
RUN go mod download
# ---------------------------------------------------------------------------------

# Kopieer nu pas de rest van de broncode (inclusief de net gebouwde frontend)
COPY --from=frontend-builder /app /go/src/github.com/evcc-io/evcc

# Bouw de uiteindelijke evcc binary (hoeft nu niet meer te downloaden, dus dit vliegt erdoorheen)
RUN go build -ldflags="-s -w" -o evcc .

# Stap 3: Het uiteindelijke compacte Home Assistant Add-on image
FROM alpine:latest
RUN apk add --no-cache tzdata ca-certificates
COPY --from=backend-builder /go/src/github.com/evcc-io/evcc/evcc /usr/local/bin/evcc

# Start EVCC automatisch op met de Home Assistant configuratie
ENTRYPOINT ["evcc"]
CMD ["--config", "/config/evcc.yaml"]
