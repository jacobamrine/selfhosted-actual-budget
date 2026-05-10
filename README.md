# Traefik Reverse Proxy Deployment

This repository contains my Docker Compose deployment for running Traefik as a reverse proxy for self-hosted Docker services.

The goal of this project is not to modify Traefik itself, but to document a clean, repeatable reverse proxy setup using Docker Compose, Portainer, Docker networking, HTTPS, Let's Encrypt, and environment-based configuration.

## Features

- Traefik reverse proxy deployed with Docker Compose
- Managed as a Portainer stack
- Docker provider with containers hidden by default
- HTTP and HTTPS entrypoints
- Let's Encrypt certificate generation using the HTTP challenge
- External Docker network for proxied services
- Traefik dashboard protected with Basic Auth
- Environment variables used for configurable values and secrets

## Repository Structure

```text
.
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

## Prerequisites

- Docker installed on the host
- Portainer or Docker Compose available for stack deployment
- A domain or subdomain pointed to the server
- Ports `80` and `443` open on the server/firewall
- An external Docker network named `proxy`

Create the proxy network if it does not already exist:

```bash
docker network create proxy
```

## Environment Variables

When deploying through Portainer, add these values as stack environment variables. For local CLI deployment, create a `.env` file from `.env.example`.

Example `.env.example`:

```env
TRAEFIK_VERSION=v3.3
ACME_EMAIL=your-email@example.com
TRAEFIK_URL=traefik.example.com
TRAEFIK_DASHBOARD_AUTH=admin:REPLACE_WITH_HASHED_PASSWORD
```

Do not commit your real `.env` file.

## Basic Auth

The Traefik dashboard is protected with Basic Auth. Generate a hashed username and password with:

```bash
htpasswd -nbB admin your-password
```

If the generated hash contains `$`, escape each `$` as `$$` when placing it in the `.env` file so Docker Compose reads it correctly.

## Deployment

This stack is deployed through Portainer using the `docker-compose.yml` file in this repository.

General deployment flow:

1. Create an external Docker network named `proxy`
2. Add the required environment variables in Portainer
3. Deploy the stack from the Compose file
4. Confirm Traefik is running and listening on ports `80` and `443`

### Optional CLI Deployment

If deploying directly from the server instead of Portainer:

```bash
docker compose up -d
docker compose logs -f traefik
docker compose down
```

## Exposing a Service

Attach the service to the same external `proxy` network and add Traefik labels.

Example:

```yaml
services:
  app:
    image: example/app:latest
    networks:
      - proxy
    labels:
      - traefik.enable=true
      - traefik.http.routers.app.rule=Host(`app.example.com`)
      - traefik.http.routers.app.entrypoints=websecure
      - traefik.http.routers.app.tls=true
      - traefik.http.routers.app.tls.certresolver=le
      - traefik.http.services.app.loadbalancer.server.port=8080

networks:
  proxy:
    external: true
```

The `loadbalancer.server.port` value should match the internal container port used by the application.

## Security Notes

- Keep `.env` out of source control
- Use a pinned Traefik version instead of `latest`
- Mount the Docker socket as read-only when possible
- Use a strong password for the dashboard
- Consider limiting dashboard access by IP address or VPN for production use

Suggested `.gitignore`:

```gitignore
.env
```

## Troubleshooting

### 404 from Traefik

Traefik is running, but no router matched the request. Check the hostname, router rule, labels, and network.

### Bad Gateway

Traefik matched the route but cannot reach the backend container. Check that the service is running, attached to the `proxy` network, and using the correct internal port.

### Certificate Issues

Check that DNS points to the server, ports `80` and `443` are open, and the ACME email is set correctly.

## What I Learned

- How to configure Traefik as a Docker reverse proxy
- How to route services using Docker labels
- How to use an external Docker network for shared reverse proxy access
- How to configure automatic HTTPS certificates with Let's Encrypt
- How to protect the Traefik dashboard with Basic Auth
- How to separate public configuration from private environment values

## Notes

This setup is intended for personal self-hosted projects and homelab-style deployments. It demonstrates Docker Compose, reverse proxy routing, HTTPS certificate management, Portainer stack deployment, and Docker networking.
