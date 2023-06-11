# Assetto Corsa

[Assetto Corsa](https://store.steampowered.com/app/244210/Assetto_Corsa/) is a realistic racing simulator. This game server has a community made server manager available on [Docker Hub](https://hub.docker.com/r/seejy/assetto-server-manager), however, I have made a small fork of this to update the source for SteamCMD since I've been having quite a lot of trouble getting it to work. This fork is available from [GitHub Packages](https://github.com/moonstar-x/assetto-server-manager/pkgs/container/assetto-server-manager), which is the image we'll use for this.

## Pre-Installation

We'll create a folder in the main user's home where all the game server's data will be saved.

```bash
mkdir ~/games/assettocorsa
```

## Docker Compose

The game server will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  assettocorsa:
    image: ghcr.io/moonstar-x/assetto-server-manager:master
    restart: unless-stopped
    user: 1000:1000
    ports:
      - 8772:8772
      - 9600:9600
      - 9600:9600/udp
      - 8081:8081
    volumes:
      - ./server:/home/assetto/server-manager/assetto
      - ./data:/home/assetto/server-manager
```

!!! warning
    Make sure to create the `data` and `server` folder before starting the container, otherwise you'll have some problems with the server data being saved.

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

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8772/tcp
sudo ufw allow 9600/udp
sudo ufw allow 9600/tcp
sudo ufw allow 8081/tcp
```

## Starting for the First Time

Start up the game server with:

```bash
docker-compose up -d
```

## Joining the Server

If you have set the server to be LAN only, you may join your server by going to the following URL:

```text
https://acstuff.ru/s/q:race/online/join?ip=<IP>&httpPort=8081
```

Make sure the clients have [AC Content Manager](https://assettocorsa.club/content-manager.html) to be able to access through that URL.
