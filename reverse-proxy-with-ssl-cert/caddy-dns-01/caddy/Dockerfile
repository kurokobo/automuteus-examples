ARG CADDY_TAG

FROM caddy:${CADDY_TAG}-builder-alpine AS builder
ARG CADDY_TAG
ARG CADDY_DNS_AZURE_TAG
RUN xcaddy build v${CADDY_TAG} \
    --with github.com/caddy-dns/azure@v${CADDY_DNS_AZURE_TAG}

FROM caddy:${CADDY_TAG}-alpine
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
