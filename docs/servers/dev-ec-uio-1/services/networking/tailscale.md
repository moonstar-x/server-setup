# Tailscale

[Tailscale](https://tailscale.com) is a virtual LAN service, similar to Hamachi, that allows you to have your services exposed through a VPN.

There is an official image for this service that we'll use: [tailscale/tailscale](https://hub.docker.com/r/tailscale/tailscale).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/networking/tailscale
```

## Getting an Auth Key

First head over to your Tailscale account's [Dashboard > Settings > Keys](https://login.tailscale.com/admin/settings/keys) and create `Generate auth key...`. You can name this key whatever you want and set the expiry to 1 day since we'll use it right off the bat.

Once you generate it, copy it and save it somewhere, we'll use it in the [docker-compose.yml](#docker-compose) file.

## Docker Compose

*Tailscale* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  vpn:
    image: tailscale/tailscale:latest
    restart: unless-stopped
    network_mode: host
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    volumes:
      - /dev/net/tun:/dev/net/tun
      - ./data:/var/lib/tailscale
    environment:
      TZ: America/Guayaquil
      TS_AUTHKEY: ${TAILSCALE_AUTH_KEY}
      TS_EXTRA_ARGS: --advertise-tags=tag:container --advertise-exit-node --accept-routes
      TS_STATE_DIR: /var/lib/tailscale
      TS_USERSPACE: false
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
TAILSCALE_AUTH_KEY=
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
