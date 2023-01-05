# Jenkins

[Jenkins](https://www.jenkins.io/) is a CI/CD service. This service has an official image available on [Docker Hub](https://hub.docker.com/r/jenkins/jenkins) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/jenkins
```

## Docker Compose

*Jenkins* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  jenkins:
    image: jenkins/jenkins:lts
    restart: unless-stopped
    user: 1000:1000
    volumes:
      - ./data:/var/jenkins_home
    ports:
      - 10800:8080
    environment:
      - TZ=America/Guayaquil
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
