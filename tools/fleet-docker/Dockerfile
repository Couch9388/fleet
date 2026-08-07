# Build stage — compiles Fleet from source
FROM golang:1.26.5-alpine AS builder
ARG FLEET_VERSION=unknown

WORKDIR /build

RUN apk add --no-cache git ca-certificates gcc musl-dev make

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=1 GOOS=linux go build -o fleet \
    -tags=full,fts5,netgo \
    -trimpath \
    -ldflags="-s -w -extldflags '-static'" \
    ./cmd/fleet

# Runtime stage
FROM alpine:3.23.4@sha256:5b10f432ef3da1b8d4c7eb6c487f2f5a8f096bc91145e68878dd4a5019afde11
ARG FLEET_VERSION=unknown
LABEL maintainer="Fleet Developers"
LABEL org.opencontainers.image.version="${FLEET_VERSION}"
LABEL org.opencontainers.image.vendor="Fleet Device Management Inc."
LABEL org.opencontainers.image.title="fleet"
LABEL org.opencontainers.image.source="https://github.com/fleetdm/fleet"
LABEL org.opencontainers.image.url="https://fleetdm.com"
LABEL org.opencontainers.image.documentation="https://fleetdm.com/docs"

RUN apk --update add ca-certificates
RUN apk --no-cache add jq
RUN apk --no-cache upgrade openssl libcrypto3 libssl3

# Create fleet group and user
RUN addgroup -S fleet && adduser -S fleet -G fleet

# Create installers directory
RUN mkdir -p /opt/fleet/installers && chown -R fleet:fleet /opt/fleet

USER fleet

COPY --from=builder /build/fleet /usr/bin/fleet

CMD ["fleet", "serve"]
