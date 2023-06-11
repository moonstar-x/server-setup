# Plausible

[Plausible](https://plausible.io/) is a Web analytics dashboard. This service has an official image available on [Docker Hub](https://hub.docker.com/r/plausible/analytics) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the analytics server's data will be saved.

```bash
mkdir ~/analytics/plausible
```

Next, create a folder for the Clickhouse configs inside this new folder:

```bash
mkdir clickhouse
```

And create the following files:

```bash
nano clickhouse/clickhouse-config.xml
```

```xml
<clickhouse>
    <logger>
        <level>warning</level>
        <console>true</console>
    </logger>

    <!-- Stop all the unnecessary logging -->
    <query_thread_log remove="remove"/>
    <query_log remove="remove"/>
    <text_log remove="remove"/>
    <trace_log remove="remove"/>
    <metric_log remove="remove"/>
    <asynchronous_metric_log remove="remove"/>
    <session_log remove="remove"/>
    <part_log remove="remove"/>
</clickhouse>
```

```bash
nano clickhouse/clickhouse-user-config.xml
```

```xml
<clickhouse>
    <profiles>
        <default>
            <log_queries>0</log_queries>
            <log_query_threads>0</log_query_threads>
        </default>
    </profiles>
</clickhouse>
```

And now change the permissions for this folder:

```bash
chmod -R 777 clickhouse
```

## Docker Compose

*Plausible* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  plausible:
    image: plausible/analytics:v1
    restart: unless-stopped
    command: sh -c "sleep 10 && /entrypoint.sh db createdb && /entrypoint.sh db migrate && /entrypoint.sh run"
    depends_on:
      - db
      - events_db
    ports:
      - 10500:80
    environment:
      - TZ=America/Guayaquil
      - BASE_URL=CHANGE_THIS_URL
      - PORT=80
      - SECRET_KEY_BASE=CHANGE_THIS_SECRET_KEY
      - DISABLE_REGISTRATION=true
      - DATABASE_URL=postgres://plausible:CHANGE_THIS_DB_PASS@db/plausible?sslmode=disable
      - CLICKHOUSE_DATABASE_URL=http://events_db:8123/plausible_events_db

  db:
    image: postgres:14-alpine
    restart: unless-stopped
    volumes:
      - ./db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=plausible
      - POSTGRES_PASSWORD=CHANGE_THIS_DB_PASS

  events_db:
    image: clickhouse/clickhouse-server:22-alpine
    restart: unless-stopped
    volumes:
      - ./events-data:/var/lib/clickhouse
      - ./clickhouse/clickhouse-config.xml:/etc/clickhouse-server/config.d/logging.xml:ro
      - ./clickhouse/clickhouse-user-config.xml:/etc/clickhouse-server/users.d/logging.xml:ro
    ulimits:
      nofile:
        soft: 262144
        hard: 262144
    environment:
      - CLICKHOUSE_DB=plausible_events_db
```

!!! note
    * Make sure to change `CHANGE_THIS_URL` with the URL of where this service will be hosted with protocol. For instance, `https://analytics.example.com`.
    * Make sure to change `CHANGE_THIS_DB_PASS` with anything you want.
    * Make sure to change `CHANGE_THIS_SECRET_KEY` with a random key. You may use the `openssl rand -hex 64` command to generate one.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 10500/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
