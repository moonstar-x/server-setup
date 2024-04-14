# PiHole

[PiHole](https://pi-hole.net/) is a DNS server that can be used primarily as a network wide ad blocker, as well as local DNS for custom queries.

There is an official image for this service that we'll use: [pihole/pihole](https://hub.docker.com/r/pihole/pihole).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/networking/pihole
```

## Docker Compose

*PiHole* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  dns:
    image: pihole/pihole:latest
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    networks:
      default:
      proxy_external:
        aliases:
          - pihole
    ports:
      - 53:53
      - 53:53/udp
    volumes:
      - ./config:/etc/pihole
      - ./dnsmasq:/etc/dnsmasq.d
    environment:
      TZ: America/Guayaquil
      WEBPASSWORD: PASSWORD
      FTLCONF_LOCAL_IPV4: LOCAL_IP
      VIRTUAL_HOST: dns.home.example.com
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.pihole.rule: Host(`dns.home.example.com`)
      traefik.http.routers.pihole.entrypoints: local-https
      traefik.http.routers.pihole.tls: true
      traefik.http.routers.pihole.tls.certresolver: le
      traefik.http.routers.pihole.service: pihole@docker
      traefik.http.services.pihole.loadbalancer.server.port: 80

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to change `PASSWORD` to a custom secret value.

!!! note
    Make sure to change `LOCAL_IP` to the local IP of your server.

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

## Post Configuration

With the service up and running, make sure to complete the following steps:

1. Head over to `Settings > DNS` and enable the option that says `Permit all origins`.
2. In `Settings > DNS`, enable `Google (ECS, DNSSEC)` and `Cloudflare (DNSSEC)` for both `IPv4` and `IPv6`.

Since we're not using *PiHole* as our DHCP server, we must edit our router's DHCP server to push the server's IP address as the primary DNS server.

- Since my server's local IP address is `192.168.0.4` and my router's IP address is `192.168.0.1`, I'll set up my DNS servers as:
  - `192.168.0.4`
  - `192.168.0.1`

If your router supports `IPv6`, make sure to make the same change for the `IPv6` DHCP.

- In the case of `IPv6`, make sure to take the local link address for your server. In my case I'll set up my DNS servers as:
  - `2800:....:....`
  - `fe80::1`

With these changes in place, any device that connects to the router will use *PiHole* as the DNS resolver.
