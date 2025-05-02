# Gitea Runners

[Gitea](https://gitea.io/en-us/) is a self-hosted git server, useful for having a private VCS solution. Gitea comes with CI support with [Act Runners](https://docs.gitea.com/usage/actions/act-runner) which essentially emulate GitHub Actions. 

There is an official image for this service that we'll use: [gitea/act_runner](https://hub.docker.com/r/gitea/act_runner/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/gitea-runners
```

## Getting a Token

In order to connect a runner with your Gitea instance you'll need a token. This token can be found by heading over to `https://my.gitea.instance/-/admin/actions/runners` or `Site Administration > Actions > Runners`. Here you'll find a button that says `Create new Runner` click on it to get the **registration token** for your instance.

## Docker Compose

*Gitea Runners* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  runner-1:
    image: gitea/act_runner:latest
    restart: unless-stopped
    ports:
      - 20000:20000
    volumes:
      - ./runner-1/data:/data
      - ./cache:/root/.cache
      - ./runner-1/config.yaml:/config.yaml
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: America/Guayaquil
      GITEA_INSTANCE_URL: ${GITEA_URL}
      GITEA_RUNNER_REGISTRATION_TOKEN: ${RUNNER_TOKEN}
      GITEA_RUNNER_NAME: local-runner-1
      CONFIG_FILE: /config.yaml

  runner-2:
    image: gitea/act_runner:latest
    restart: unless-stopped
    ports:
      - 20010:20010
    volumes:
      - ./runner-2/data:/data
      - ./cache:/root/.cache
      - ./runner-2/config.yaml:/config.yaml
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: America/Guayaquil
      GITEA_INSTANCE_URL: ${GITEA_URL}
      GITEA_RUNNER_REGISTRATION_TOKEN: ${RUNNER_TOKEN}
      GITEA_RUNNER_NAME: local-runner-2
      CONFIG_FILE: /config.yaml

  runner-3:
    image: gitea/act_runner:latest
    restart: unless-stopped
    ports:
      - 20020:20020
    volumes:
      - ./runner-3/data:/data
      - ./cache:/root/.cache
      - ./runner-3/config.yaml:/config.yaml
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: America/Guayaquil
      GITEA_INSTANCE_URL: ${GITEA_URL}
      GITEA_RUNNER_REGISTRATION_TOKEN: ${RUNNER_TOKEN}
      GITEA_RUNNER_NAME: local-runner-3
      CONFIG_FILE: /config.yaml
```

!!! note
    I've made 3 containers with the same image. It's up to you if you want multiple or if only one is enough.

### Secrets

Make sure to create a `.env` file with the following structure:

```text
GITEA_URL=

RUNNER_TOKEN=
```

## Configuration

Make sure to create the `runner` folders according to the volume defined in each container and inside each of these folders create a `config.yaml` file with the following content:

```yaml
cache:
  enabled: true
  dir: ''
  host: LAN_HOST
  port: 20000
```

> Replace `LAN_HOST` with the LAN IP of your server. (i.e. `10.0.0.22` or `192.168.100.22).

Additionally, make sure to allow the port you've defined in the config through your firewall:

```bash
sudo ufw allow 20000/tcp
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
