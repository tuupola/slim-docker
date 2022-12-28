FROM caddy:2.6-builder AS builder
RUN xcaddy build \
    --with github.com/lucaslorentz/caddy-docker-proxy@v2.8.1

FROM caddy:2.6
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
ENTRYPOINT ["/usr/bin/caddy"]
CMD ["docker-proxy"]
