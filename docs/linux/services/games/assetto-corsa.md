# Assetto Corsa

[Assetto Corsa](https://store.steampowered.com/app/244210/Assetto_Corsa/) is a realistic racing simulator.

This game server has a community made server manager available on [Docker Hub](https://hub.docker.com/r/seejy/assetto-server-manager), however, I have made a small fork of this to update the source for SteamCMD since I've been having quite a lot of trouble getting it to work.

This fork is available on [ghcr.io/moonstar-x/assetto-server-managers](https://github.com/moonstar-x/assetto-server-manager/pkgs/container/assetto-server-manager), which is the image we'll use for this.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/games/assetto-corsa
```

## Docker Compose

*Assetto Corsa* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/moonstar-x/assetto-server-manager:master
    restart: unless-stopped
    user: 1000:1000
    networks:
      default:
      proxy_external:
        aliases:
          - assetto_corsa
    ports:
      - 9600:9600
      - 9600:9600/udp
      - 8081:8081
    volumes:
      - ./server:/home/assetto/server-manager/assetto
      - ./data:/home/assetto/server-manager
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.assetto_corsa.rule: Host(`subdomain.example.com`)
      traefik.http.routers.assetto_corsa.entrypoints: public
      traefik.http.routers.assetto_corsa.service: assetto_corsa@docker
      traefik.http.services.assetto_corsa.loadbalancer.server.port: 8772

networks:
  proxy_external:
    external: true
```

!!! warning
    Make sure to create the `data` and `server` folder before starting the container, otherwise you'll have some problems with the server data being saved.

!!! note
    In the case of the `user` directive, `1000:1000` corresponds to the user's `UID:GID`. You can find the values for your own user by running `id $whoami`.

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Configuration

Create a config file inside `data/config.yml`:

```bash
nano data/config.yml
```

And paste the following:

```yaml
steam:
  username: STEAM_USER
  password: STEAM_PASS
  install_path: assetto
  executable_path: acServer
  force_update: false

http:
  hostname: 0.0.0.0:8772
  session_key: RANDOMLY_GENERATE_THIS
  server_manager_base_URL:
  session_store_type: cookie
  session_store_path: ''

  tls:
    enabled: false
    cert_path: ''
    key_path: ''

monitoring:
  enabled: true

store:
  type: boltdb
  path: server_manager.db
  shared_data_path:
  scheduled_event_check_loop: 0s

accounts:
  admin_password_override:

live_map:
  refresh_interval_ms: 500

server:
  audit_logging: true
  performance_mode: false
  dont_open_browser: false
  scan_content_folder_for_chanes: true
  use_car_name_cache: true
  persist_mid_session_results: false
  plugins:

championships:
  recaptcha:
    site_key:
    secret_key:

lua:
  enabled: false
```

!!! note
    Make sure to replace `STEAM_USER` and `STEAM_PASS` with your steam account's information. I recommend you create a separate Steam account **with Steam Guard disabled**. You don't need an Assetto Corsa license to download the dedicated server.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.

## Joining the Server

If you have set the server to be LAN only, you may join your server by going to the following URL:

```text
https://acstuff.ru/s/q:race/online/join?ip=<IP>&httpPort=8081
```

Make sure the clients have [AC Content Manager](https://assettocorsa.club/content-manager.html) to be able to access through that URL.
