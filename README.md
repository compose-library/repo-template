# Docker Compose for Example Service

## Usage
- Copy sample.env to .env and update values

## To add SSL using Traefik
- Add `DOMAIN` and `ACME_EMAIL` to .env
- Add Traefik to the services block in docker-compose.yml:

```yaml
traefik:
	image: "traefik:alpine"
	ports:
	  - "80:80"
	  - "443:443"
	volumes:
	  - "./letsencrypt:/letsencrypt"
	  - "/var/run/docker.sock:/var/run/docker.sock:ro"
	restart: always
	command:
	  - "--providers.docker=true"
	  - "--providers.docker.exposedbydefault=false"
	  - "--entrypoints.web.address=:80"
	  - "--entrypoints.websecure.address=:443"
	  - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
	  - "--certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL}"
	  - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
```

- Add the label to your service which you want to be served through Traefik's proxy, e.g.:

```yaml
version: "3.5"

services:
  frontend:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.sample.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.sample.entrypoints=websecure,web"
      - "traefik.http.routers.sample.tls.certresolver=letsencrypt"
```
- Remove any ports directly mapped to the service

- *To add http to https redirection you'll need to add [labels]*(https://github.com/docker-compositions/traefik/blob/main/docker-compose.yml)