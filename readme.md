# Traefik v2 HTTPS SSL Localhost

This project provides a Docker Compose template for setting up Traefik v2 with HTTPS and SSL support for localhost. It also includes an example template for setting up multiple Postgres connections.

## Setup

To get started, create a Docker network using the following command:

```sh
# Create a Docker network
docker network create traefik
```

You will also need to generate SSL certificates for the domain "docker.localhost" and "domain.local" along with their sub-domains. You can use [mkcert](https://github.com/FiloSottile/mkcert) to generate the certificates. Install mkcert using the command `mkcert -install` and then generate the certificates using the following command:

```sh
# Generate certificates
mkcert -cert-file certs/local-cert.pem -key-file certs/local-key.pem "localhost" "*.localhost" "localhost"
```

Make sure to place the generated certificates in the `certs` directory of this project.

## Multiple Postgres Connections

You can use the provided Docker Compose template to set up multiple Postgres connections. Here's an example template for the Postgres service:

```yaml
services:
  db:
    hostname: ${POSTGRES_HOST}
    image: postgres:12.0-alpine
    restart: unless-stopped
  
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-user}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-changeme}
      POSTGRES_DB: ${POSTGRES_DB:-db}

    labels:
      - traefik.enable=true
      - traefik.tcp.routers.postgres.tls=true
      - traefik.tcp.routers.postgres.rule=HostSNI(`${DB_HOST}`)
      - traefik.tcp.routers.postgres.entrypoints=postgres
      - traefik.tcp.services.postgres.loadbalancer.server.port=5432
  
    expose:
      - 5432

    networks:
      - traefik
```

This template sets up a Postgres service with Traefik labels for enabling HTTPS, SSL, and routing based on the Host SNI (Server Name Indication) header. It also exposes port 5432 and connects to the `traefik` Docker network.

Don't forget to add the `$POSTGRES_HOST` to your `/etc/hosts` file.

