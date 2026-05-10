# Actual Budget Self-Hosted Deployment

This repository contains my Docker Compose deployment for self-hosting [Actual Budget](https://actualbudget.org/), an open-source, privacy-focused personal finance application.

The goal of this project is not to modify Actual Budget itself, but to document a clean, repeatable self-hosted deployment using Docker Compose, Portainer, Traefik, HTTPS, and persistent storage.

- Actual Budget website: https://actualbudget.org/
- Actual Budget GitHub: https://github.com/actualbudget/actual

## Deployment Notes

- Deployed as a Portainer stack
- Runs Actual Budget using Docker Compose
- Routed through a Traefik reverse proxy
- Secured with HTTPS using Let's Encrypt
- Uses persistent storage for application data
- Keeps configurable values in environment variables

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
- Traefik reverse proxy already configured
- External Docker network named `proxy`
- A domain or subdomain pointed to the server

Create the proxy network if it does not already exist:

```bash
docker network create proxy
```

## Environment Variables

When deploying through Portainer, add the required values as stack environment variables.

For local CLI deployment, create a `.env` file from `.env.example`:

```bash
cp .env.example .env
```

Example `.env.example`:

```env
ACTUAL_VERSION=latest
ACTUAL_URL=actual.example.com
```

Do not commit your real `.env` file.

## Example Service Configuration

Actual should be attached to the same external `proxy` network used by Traefik. Traefik labels route traffic from the configured hostname to the Actual container.

Example label pattern:

```yaml
labels:
  - traefik.enable=true
  - traefik.http.routers.actual.rule=Host(`${ACTUAL_HOST}`)
  - traefik.http.routers.actual.entrypoints=websecure
  - traefik.http.routers.actual.tls=true
  - traefik.http.routers.actual.tls.certresolver=le
  - traefik.http.services.actual.loadbalancer.server.port=5006
```

The `loadbalancer.server.port` value should match the internal port used by the Actual container.

## Start the Stack

If deploying through Portainer, deploy this repository as a stack and provide the required environment variables.

For direct Docker Compose deployment:

```bash
docker compose up -d
```

View logs:

```bash
docker compose logs -f
```

Stop the stack:

```bash
docker compose down
```

## Updating

Update the image version, then redeploy the stack through Portainer.

For CLI deployment:

```bash
docker compose pull
docker compose up -d
```

Using a pinned version instead of `latest` is recommended for more predictable updates.

## Security Notes

- Keep `.env` out of source control
- Use HTTPS when exposing Actual outside the local network
- Keep the application data volume persistent and backed up securely
- Limit public exposure if the service is only intended for personal use

Suggested `.gitignore`:

```gitignore
.env
data/
```

## Troubleshooting

### Site does not load

Check DNS, Traefik labels, HTTPS certificate status, and whether the container is attached to the `proxy` network.

### Bad Gateway

Traefik matched the route but cannot reach the Actual container. Check that the container is running and that the internal service port is correct.

### Data does not persist

Confirm that the Compose file uses a persistent Docker volume or bind mount for Actual's data directory.

## What I Learned

- How to deploy a self-hosted application with Docker Compose
- How to manage a Portainer stack
- How to route an application through Traefik
- How to use HTTPS with a self-hosted service
- How to manage persistent application data
- How to separate public configuration from private environment values

## Notes

This is a personal infrastructure/deployment project. Actual Budget is developed and maintained by the Actual Budget project.
