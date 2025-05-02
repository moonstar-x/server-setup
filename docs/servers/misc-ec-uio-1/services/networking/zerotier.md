# ZeroTier-One

[ZeroTier-One](https://www.zerotier.com/) is a virtual LAN service, similar to Hamachi, that allows you to have your services exposed through a VPN.

There is an official image for this service that we'll use: [zerotier/zerotier](https://hub.docker.com/r/zerotier/zerotier).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/networking/zerotier
```

## Creating a Network

To create a network, simply visit [My ZeroTier](https://my.zerotier.com/), login to your account (or create one if needed) and simply click on the `Create Network` button. This will give you a **Network ID** (which you should keep since we'll need this). This **Network ID** is what you need to share with your friends so that they can connect to your network. If you leave the network settings to be private, you may need to manually authorize new members into the network.

## Docker Compose

*ZeroTier-One* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  vpn:
    image: zerotier/zerotier:latest
    restart: unless-stopped
    network_mode: host
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    volumes:
      - ./config/authtoken.secret:/var/lib/zerotier-one/authtoken.secret
      - ./config/identity.public:/var/lib/zerotier-one/identity.public
      - ./config/identity.secret:/var/lib/zerotier-one/identity.secret
    command: ${NETWORK_ID}
    environment:
      TZ: America/Guayaquil
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
NETWORK_ID=
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
