# Samba

*Samba* lets your Linux based server share files and folders on a Windows File Sharing Workgroup using the same protocol (SMB/CIFS), this is pretty useful when you need to share files between computers on your network. This also helps to allow your files to be accessed through the Internet (although it should only be done through a VPN for security purposes).

There is no official image for this service, so we'll use [ghcr.io/servercontainers/samba](https://github.com/ServerContainers/samba).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/networking/samba
```

## Docker Compose

*Samba* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  server:
    image: ghcr.io/servercontainers/samba:smbd-only-latest
    restart: unless-stopped
    network_mode: host
    volumes:
      - /media:/media
      - ~/:/home
    environment:
      TZ: America/Guayaquil
      SAMBA_CONF_LOG_LEVEL: 3
      AVAHI_DISABLE: disabled
      WSDD2_DISABLE: disabled
      ACCOUNT_foo: ${ACCOUNT_FOO}
      UID_foo: ${UID_FOO}
      SAMBA_VOLUME_CONFIG_downloads: ${CONFIG_DOWNLOADS}
```

Feel free to edit the volumes and the volume configs at the bottom to your liking.

### Secrets

Make sure to create a `.env` file with the following structure:

```text
ACCOUNT_FOO=
UID_FOO=

CONFIG_DOWNLOADS=
```

For the `ACCOUNT_FOO` variable, you will need to run the following command, which will generate a user hash that should be placed in the `.env` file.

```bash
docker run -it --rm ghcr.io/servercontainers/samba:smbd-only-latest create-hash.sh
```

This will generate a hash that you can use instead of adding your password in plaintext.

The shares are defined through environment variables prefixed by `SAMBA_VOLUME_CONFIG`, you can create as many as you need.
Same story for `ACCOUNT` and `UID` which are used to define accounts with access to the share and their respective UID.

For the `CONFIG_*` variables, feel free to base yourself of the following example:

```text
[downloads]
  comment = Downloads directory.
  path = /media/sata_2tb/Downloads
  available = yes
  read only = no
  browsable = yes
  writeable = yes
  create mask = 0744
  directory mask = 0755
  force user = foo
  force group = foo
  public = no
  guest ok = no
```

!!! note
    Make sure the wrap each block with double-quotes (`"`) in the `.env` file to support multiline variables.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 455/tcp
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
