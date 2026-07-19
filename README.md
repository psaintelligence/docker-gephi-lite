# docker-gephi-lite

A Docker container that serves [**Gephi Lite**](https://github.com/gephi/gephi-lite) - the
open-source graph exploration and visualization tool - as static assets rooted at
**`/`**, so it drops cleanly behind a reverse proxy or alongside other web apps on
the same host.

Maintained by **[PSA Intelligence](https://github.com/psaintelligence)**.

## Features

Gephi Lite runs entirely in the browser - this container only ships the compiled
static build with nginx. There is no server-side processing, persistence layer, or
graph compute endpoint.

- **Root-served** - served from `/` so it can sit behind any reverse proxy without
  path rewriting.
- **Static hosting** - shipped as a single nginx container, trivially deployable
  behind any reverse proxy or load balancer.
- **Pinned upstream releases** - each image tag is built from a specific
  `gephi/gephi-lite` release for reproducibility.

## Quick start

Images are published to both Docker Hub and the GitHub Container Registry (GHCR).
Pick whichever registry you prefer - the image content is identical.

```bash
# Docker Hub
docker pull psaintelligence/gephi-lite:latest

# GHCR
docker pull ghcr.io/psaintelligence/gephi-lite:latest

# Run on http://localhost:8080/
docker run -d \
  --name gephi-lite \
  -p 8080:80 \
  --restart unless-stopped \
  psaintelligence/gephi-lite:latest
```

Open <http://localhost:8080/> in your browser.

### Docker Compose

```yaml
services:
  gephi-lite:
    image: ghcr.io/psaintelligence/gephi-lite:latest  # or psaintelligence/gephi-lite:latest
    ports:
      - "8080:80"
    restart: unless-stopped
```

```bash
docker compose up -d
```

### Serving behind a reverse proxy

Because the app is rooted at `/`, you can place it behind any front proxy. Example
with nginx upstream:

```nginx
location / {
    proxy_pass http://gephi-lite:80/;
    proxy_set_header Host              $host;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

## Container Repos
- Docker Hub - <https://hub.docker.com/r/psaintelligence/gephi-lite>
- GHCR - <https://github.com/psaintelligence/docker-gephi-lite/pkgs/container/gephi-lite>

## Building from source

This repository clones the upstream `gephi/gephi-lite` source at a pinned tag,
runs `npm install` + `npm run build` with `BASE_URL=/`, then ships the compiled
static output with nginx.

```bash
git clone https://github.com/psaintelligence/docker-gephi-lite.git
cd docker-gephi-lite

# Build with the default upstream tag (@gephi/gephi-lite@1.0.2)
docker build -t gephi-lite:local .

# Build against a different upstream tag
docker build -t gephi-lite:local \
  --build-arg COMMIT_TAG='@gephi/gephi-lite@1.0.2' .

docker run --rm -p 8080:80 gephi-lite:local
```

See the [`Dockerfile`](./Dockerfile) for the full multi-stage build definition.

## Related image - `gephi-lite-path`

An additional Docker image, **`gephi-lite-path`**, is maintained in the sibling
repository [`docker-gephi-lite-path`](https://github.com/psaintelligence/docker-gephi-lite-path).
It serves the same Gephi Lite build rooted at **`/gephi-lite/`** instead of `/`,
making it suitable for hosting alongside other web apps on the same host without
path conflicts.

- Container source - <https://github.com/psaintelligence/docker-gephi-lite-path>
- Docker Hub - <https://hub.docker.com/r/psaintelligence/gephi-lite-path>
- GHCR - <https://github.com/psaintelligence/docker-gephi-lite-path/pkgs/container/gephi-lite-path>

```bash
docker pull psaintelligence/gephi-lite-path:latest
# Served at http://localhost:8080/gephi-lite/
```

## Credits

- **Upstream project** - [`gephi/gephi-lite`](https://github.com/gephi/gephi-lite).

## License

GNU General Public License v3.0 - see [`LICENSE`](./LICENSE).
