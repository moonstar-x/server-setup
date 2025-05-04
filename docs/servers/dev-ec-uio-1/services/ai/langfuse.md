# Langfuse

[Langfuse](https://langfuse.com) is a self-hosted tracing tool for LLMs.

There is an official image for this service that we'll use: [langfuse/langfuse](https://hub.docker.com/r/langfuse/langfuse/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/ai/langfuse
```

## Docker Compose

*Langfuse* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  worker:
    image: langfuse/langfuse-worker:3
    restart: unless-stopped
    depends_on: &langfuse-depends-on
      postgres:
        condition: service_healthy
      minio:
        condition: service_healthy
      redis:
        condition: service_healthy
      clickhouse:
        condition: service_healthy
    environment: &langfuse-worker-env
      TZ: America/Guayaquil
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DATABASE}
      SALT: ${SALT}
      ENCRYPTION_KEY: ${ENCRYPTION_KEY}
      TELEMETRY_ENABLED: false
      LANGFUSE_ENABLE_EXPERIMENTAL_FEATURES: true
      AUTH_DISABLE_SIGNUP: true
      CLICKHOUSE_MIGRATION_URL: clickhouse://clickhouse:9000
      CLICKHOUSE_URL: http://clickhouse:8123
      CLICKHOUSE_USER: ${CLICKHOUSE_USER}
      CLICKHOUSE_PASSWORD: ${CLICKHOUSE_PASSWORD}
      CLICKHOUSE_CLUSTER_ENABLED: false
      REDIS_HOST: redis
      REDIS_PORT: 6379
      REDIS_AUTH: ${REDIS_AUTH}
      REDIS_TLS_ENABLED: false
      LANGFUSE_S3_EVENT_UPLOAD_BUCKET: ${MINIO_BUCKET}
      LANGFUSE_S3_EVENT_UPLOAD_REGION: ${MINIO_REGION}
      LANGFUSE_S3_EVENT_UPLOAD_ACCESS_KEY_ID: ${MINIO_ROOT_USER}
      LANGFUSE_S3_EVENT_UPLOAD_SECRET_ACCESS_KEY: ${MINIO_ROOT_PASSWORD}
      LANGFUSE_S3_EVENT_UPLOAD_ENDPOINT: http://minio:9000
      LANGFUSE_S3_EVENT_UPLOAD_FORCE_PATH_STYLE: true
      LANGFUSE_S3_EVENT_UPLOAD_PREFIX: events/
      LANGFUSE_S3_MEDIA_UPLOAD_BUCKET: ${MINIO_BUCKET}
      LANGFUSE_S3_MEDIA_UPLOAD_REGION: ${MINIO_REGION}
      LANGFUSE_S3_MEDIA_UPLOAD_ACCESS_KEY_ID: ${MINIO_ROOT_USER}
      LANGFUSE_S3_MEDIA_UPLOAD_SECRET_ACCESS_KEY: ${MINIO_ROOT_PASSWORD}
      LANGFUSE_S3_MEDIA_UPLOAD_ENDPOINT: https://${DOMAIN_MINIO}
      LANGFUSE_S3_MEDIA_UPLOAD_FORCE_PATH_STYLE: true
      LANGFUSE_S3_MEDIA_UPLOAD_PREFIX: media/
      LANGFUSE_S3_BATCH_EXPORT_ENABLED: false
      LANGFUSE_S3_BATCH_EXPORT_BUCKET: ${MINIO_BUCKET}
      LANGFUSE_S3_BATCH_EXPORT_REGION: ${MINIO_REGION}
      LANGFUSE_S3_BATCH_EXPORT_PREFIX: exports/
      LANGFUSE_S3_BATCH_EXPORT_ENDPOINT: http://minio:9000
      LANGFUSE_S3_BATCH_EXPORT_EXTERNAL_ENDPOINT: https://${DOMAIN_MINIO}
      LANGFUSE_S3_BATCH_EXPORT_ACCESS_KEY_ID: ${MINIO_ROOT_USER}
      LANGFUSE_S3_BATCH_EXPORT_SECRET_ACCESS_KEY: ${MINIO_ROOT_PASSWORD}
      LANGFUSE_S3_BATCH_EXPORT_FORCE_PATH_STYLE: true

  web:
    image: langfuse/langfuse:3
    restart: unless-stopped
    depends_on: *langfuse-depends-on
    networks:
      default:
      proxy_external:
        aliases:
          - langfuse
    expose:
      - 3000
    environment:
      <<: *langfuse-worker-env
      NEXTAUTH_URL: https://${DOMAIN_LANGFUSE}
      NEXTAUTH_SECRET: ${NEXTAUTH_SECRET}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.langfuse.rule: Host(`${DOMAIN_LANGFUSE}`)
      traefik.http.routers.langfuse.entrypoints: public
      traefik.http.routers.langfuse.service: langfuse@docker
      traefik.http.services.langfuse.loadbalancer.server.port: 3000

  minio:
    image: minio/minio:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - langfuse-minio
    healthcheck:
      test: ["CMD", "mc", "ready", "local"]
      interval: 1s
      timeout: 5s
      retries: 5
      start_period: 1s
    entrypoint: sh
    command: -c 'mkdir -p /data/langfuse && minio server --address ":9000" --console-address ":9001" /data'
    volumes:
      - ./minio:/data
    environment:
      TZ: America/Guayaquil
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.langfuse-minio.rule: Host(`${DOMAIN_MINIO}`)
      traefik.http.routers.langfuse-minio.entrypoints: public
      traefik.http.routers.langfuse-minio.service: langfuse-minio@docker
      traefik.http.services.langfuse-minio.loadbalancer.server.port: 9000

  postgres:
    image: postgres:17
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 3s
      timeout: 3s
      retries: 10
    volumes:
      - ./postgres:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: ${POSTGRES_DATABASE}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

  redis:
    image: redis:7
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 3s
      timeout: 10s
      retries: 10
    command: --requirepass ${REDIS_AUTH}
    environment:
      TZ: America/Guayaquil

  clickhouse:
    image: clickhouse/clickhouse-server:latest
    restart: unless-stopped
    healthcheck:
      test: wget --no-verbose --tries=1 --spider http://localhost:8123/ping || exit 1
      interval: 5s
      timeout: 5s
      retries: 10
      start_period: 1s
    volumes:
      - ./clickhouse/data:/var/lib/clickhouse
      - ./clickhouse/logs:/var/log/clickhouse-server
    environment:
      TZ: America/Guayaquil
      CLICKHOUSE_DB: ${CLICKHOUSE_DATABASE}
      CLICKHOUSE_USER: ${CLICKHOUSE_USER}
      CLICKHOUSE_PASSWORD: ${CLICKHOUSE_PASSWORD}

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_LANGFUSE=
DOMAIN_MINIO=

POSTGRES_DATABASE=
POSTGRES_USER=
POSTGRES_PASSWORD=

REDIS_AUTH=

CLICKHOUSE_DATABASE=
CLICKHOUSE_USER=
CLICKHOUSE_PASSWORD=

MINIO_ROOT_USER=
MINIO_ROOT_PASSWORD=
MINIO_BUCKET=
MINIO_REGION=

SALT=
ENCRYPTION_KEY=
NEXTAUTH_SECRET=
```

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
