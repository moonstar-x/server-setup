# ZeroTier-One

[ZeroTier-One](https://www.zerotier.com/) is a virtual LAN service, similar to Hamachi, that allows you to have your services exposed through a VPN. This service has an official image available on [Docker Hub](https://hub.docker.com/r/zerotier/zerotier) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/games/zerotier
```

## Creating a Network

To create a network, simply visit [My ZeroTier](https://my.zerotier.com/), login to your account (or create one if needed) and simply click on the `Create Network` button. This will give you a **Network ID** (which you should keep since we'll need this). This **Network ID** is what you need to share with your friends so that they can connect to your network. If you leave the network settings to be private, you may need to manually authorize new members into the network.

## Docker Compose

The bot will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  zerotier:
    image: zerotier/zerotier:latest
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    network_mode: host
    devices:
      - /dev/net/tun:/dev/net/tun
    command: CHANGE_THIS
    volumes:
      - ./config/authtoken.secret:/var/lib/zerotier-one/authtoken.secret
      - ./config/identity.public:/var/lib/zerotier-one/identity.public
      - ./config/identity.secret:/var/lib/zerotier-one/identity.secret
    environment:
      - TZ=America/Guayaquil
```

!!!note
    Replace `CHANGE_THIS` with your **Network ID**.

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
