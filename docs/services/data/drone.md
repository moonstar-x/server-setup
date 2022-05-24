# Drone

!!! danger "Warning"
    This service requires [Gitea](gitea.md), you must set that up before continuing with this one.

[Drone](https://drone.io) is a self-hosted CI/CD pipeline service that can be used with [Gitea](gitea.md). This service has an official image on [Docker Hub](https://hub.docker.com/r/drone/drone/) which we'll use.

## Pre-Requisites

### Gitea OAuth2 Token

First, we'll need to log into our **Gitea** instance and then head over to the **Settings** page and click on the **Applications** tab.

In here, create a new OAuth2 application. You can choose whatever name you want but the `Redirect URI` should be:

```text
https://ci.example.com/login
```

!!! note
    Replace `ci.example.com` with the actual domain where your *Drone* instance will be hosted in.

Once you've done this, you'll need to keep the `Client ID` and `Client Secret` generated, which will be necessary for the Docker Compose file.

### Shared Secret

You will also need a shared secret which will be necessary to link your runners to the main *Drone* service. You can generate a random secret by running the following command anywhere:

```bash
openssl rand -hex 16
```

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/drone
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  drone:
    image: drone/drone:latest
    restart: unless-stopped
    ports:
      - 3080:80
    volumes:
      - ./data:/data
    environment:
      - TZ=America/Guayaquil
      - DRONE_GITEA_SERVER=https://ci.example.com
      - DRONE_GITEA_CLIENT_ID=OAUTH_CLIENT_ID
      - DRONE_GITEA_CLIENT_SECRET=OAUTH_CLIENT_SECRET
      - DRONE_RPC_SECRET=SHARED_SECRET
      - DRONE_SERVER_HOST=ci.example.com
      - DRONE_SERVER_PROTO=https
      - DRONE_REGISTRATION_CLOSED=true

  runner:
    image: drone/drone-runner-docker:latest
    restart: unless-stopped
    depends_on:
      - drone
    ports:
      - 3100:3000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=America/Guayaquil
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone
      - DRONE_RPC_SECRET=SHARED_SECRET
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=local-runner
```

!!! note
    Make sure to replace the following values:
    
    + `ci.example.com` with the domain where your *Drone* instance will be hosted.
    + `OAUTH_CLIENT_ID` with the client ID generated previously.
    + `OAUTH_CLIENT_SECRET` with the client secret generated previously.
    + `SHARED_SECRET` with the shared secret generated previously.

    You may also replace `https` with `http` in the `drone` service in `DRONE_GITEA_SERVER` and `DRONE_SERVER_PROTO` to host the service over HTTP.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 3080/tcp
sudo ufw allow 3100/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
