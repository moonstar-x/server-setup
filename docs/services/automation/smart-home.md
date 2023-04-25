# Smart Home

!!! note
    This will probably become a multi-service stack in the future.

[Homebridge](https://homebridge.io/) is an IoT bridge for HomeKit that brings support to iOS's HomeKit to smart home devices that originally do not provide support for it. This service has an official image available on [Docker Hub](https://hub.docker.com/r/oznu/homebridge/) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the automation server's data will be saved.

```bash
mkdir ~/automation/smart-home
```

## Docker Compose

*Smart Home* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  service:
    image: oznu/homebridge:latest
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./homebridge:/homebridge
    environment:
      - TZ=America/Guayaquil
```

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8581/tcp
sudo ufw allow 51845/tcp
sudo ufw allow 51845/udp
```

!!! note
    Make sure to check the `51845` port applies for you, I believe this port is chosen at random when setting-up *Homebridge*. You can check this by reading
    the output logs when setting-up this service.

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
