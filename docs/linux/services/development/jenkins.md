# Jenkins

[Jenkins](https://www.jenkins.io/) is a CI/CD service.

There is an official image for this service that we'll use: [jenkins/jenkins](https://hub.docker.com/r/jenkins/jenkins).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/jenkins
```

## Agent Dockerfile

We'll use a custom `Dockerfile` to create an agent that has the necessary dependencies installed.

First, create a new folder for the agent's data.

```bash
mkdir agent
```

Then, we'll generate an SSH key that *Jenkins* will use to connect to the agent.

```bash
ssh-keygen agent/jenkins_agent_key
```

Now, run the following command:

```bash
id
```

And check for the ID of the `docker` group. In this case, this value is **998**.

Next, we'll add a `Dockerfile` for the agent:

```docker
FROM jenkins/ssh-agent:jdk11

USER root

RUN groupadd -g 998 docker

RUN apt-get update -qq
RUN apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
RUN echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update -qq && apt-get -y install docker-ce

RUN usermod -aG docker jenkins
```

Make sure to add the generated private key as a Credential inside *Jenkins* as an SSH Username with private key with `jenkins` as the username.

## Docker Compose

*Jenkins* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: jenkins/jenkins:lts
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - jenkins
    volumes:
      - ./data:/var/jenkins_home
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.jenkins.rule: Host(`subdomain.example.com`)
      traefik.http.routers.jenkins.entrypoints: public
      traefik.http.routers.jenkins.service: jenkins@docker
      traefik.http.services.jenkins.loadbalancer.server.port: 8080

  agent:
    build: ./agent/
    restart: unless-stopped
    depends_on:
      - web
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: America/Guayaquil
      JENKINS_AGENT_SSH_PUBKEY: PUBKEY_HERE

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to replace `PUBKEY_HERE` with the content of the public key generated previously.

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
