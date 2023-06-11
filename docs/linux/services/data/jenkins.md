# Jenkins

[Jenkins](https://www.jenkins.io/) is a CI/CD service. This service has an official image available on [Docker Hub](https://hub.docker.com/r/jenkins/jenkins) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/jenkins
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
version: "3.9"

services:
  jenkins:
    image: jenkins/jenkins:lts
    restart: unless-stopped
    volumes:
      - ./data:/var/jenkins_home
    ports:
      - 10800:8080
    environment:
      - TZ=America/Guayaquil

  agent:
    build: ./agent/
    restart: unless-stopped
    depends_on:
      - jenkins
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=America/Guayaquil
      - JENKINS_AGENT_SSH_PUBKEY=ssh-rsa SECRET_REPLACE_THIS
```

## Post-Installation

To avoid permission issues, create the `data` folder that will be used as a volume:

```bash
mkdir data
```

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 10800/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
