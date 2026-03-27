# Build stage: clone elastic/opentelemetry-collector-components, add our manifest, build via repo Makefile
# See https://github.com/elastic/opentelemetry-collector-components/blob/main/README.md
ARG ELASTIC_OC_VERSION=main
FROM golang:1.25-alpine AS builder
ARG ELASTIC_OC_VERSION=main
RUN apk add --no-cache git ca-certificates make bash

# Clone elastic opentelemetry-collector-components
WORKDIR /src
RUN git clone --depth 1 --branch "${ELASTIC_OC_VERSION}" \
    https://github.com/elastic/opentelemetry-collector-components.git repo
WORKDIR /src/repo

# Overwrite manifest with EDOT + awss3 + redaction (must be copied from build context)
COPY manifest.yaml distributions/elastic-components/manifest.yaml

# Build using the repo's Makefile: builds builder from internal/tools, then runs it (same as README)
RUN TARGET_GOOS=linux make genelasticcol

# Runtime stage: minimal image with the binary and default config
FROM alpine:3.21 AS certs
RUN apk --update add ca-certificates

FROM scratch
ARG USER_UID=10001
USER ${USER_UID}

COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /src/repo/_build/elastic-collector-components /elastic-collector-components
COPY --from=builder /src/repo/distributions/elastic-components/config.yaml /etc/elastic-collector-components/config.yaml

ENTRYPOINT ["/elastic-collector-components"]
CMD ["--config", "/etc/elastic-collector-components/config.yaml"]
EXPOSE 4317 4318 55678 55679
