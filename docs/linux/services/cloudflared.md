# Cloudflared (Native)

*Cloudflared* is a daemon that allows to manage [Cloudflare Argo Tunnel](https://www.cloudflare.com/products/argo-smart-routing/). We'll use this to expose our web services through Cloudflare without the need to port forward and expose our home network directly.

## Installation

To install *Cloudflared*:

```bash
wget -q https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.deb
sudo dpkg -i cloudflared-stable-linux-amd64.deb
```

## Authenticating

We now need to authenticate with Cloudflare, for this run:

```bash
cloudflared tunnel login
```

A link will be displayed on the console. Visit it on any browser, login and select the zone you wish to enable Argo Tunnel for.

This will create a `cert.pem` file inside `~/.cloudflared`.

## Creating a Tunnel

We can create as many tunnels as we want. Generally one tunnel per DNS zone should suffice. To create a tunnel, run:

```bash
cloudflared tunnel create <NAME>
```

!!! note
    Replace `<NAME>` with the desired name for your tunnel.

Once done, this will create a `.json` file inside `~/.cloudflared` that contains the tunnel credentials.

## Pre-Configuration

Since we'll create two different tunnels for two different domains, we'll create a folder in `~/.cloudflared` with the name of the tunnel lower-cased, where we'll move `cert.pem` and `UUID.json` files.

## Configuration

Inside `~/.cloudflared/<name>` we'll create a `config.yml` file with the configuration of our tunnel.

```yaml
tunnel: TUNNEL_ID
credentials-file: /home/USER/.cloudflared/NAME/TUNNEL_ID.json

ingress:
  - hostname: subdomain.domain.com
    service: http://localhost:port
  - hostname: subdomain.domain.com
    service: http://localhost:port
  - service: http_status:404
```

!!! note
    You can add as many hostname/service pairs as you want.

## Running

You can run the tunnel with:

```bash
cloudflared tunnel --config ~/.cloudflared/name/config.yml --origincert ~/.cloudflared/name/cert.pem run
```

## Auto-starting

To auto-start the tunnel, create a service file:

```bash
sudo nano /etc/systemd/system/cloudflared_name.service
```

And add the following:

```text
[Unit]
Description=Cloudflared Tunnel for $NAME
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=/home/$USER/.cloudflared/$NAME
ExecStart=/usr/local/bin/cloudflared tunnel --config ./config.yml --origincert ./cert.pem run
Restart=always

[Install]
WantedBy=multi-user.target
```

!!! note
    Replace `$USER` and `$NAME` with the correct values for your tunnel.

Once saved, start and enable the service.

```bash
sudo systemctl start cloudflared_name.service
sudo systemctl enable cloudflared_name.service
```

## Last Words

Repeat the process from [Authenticating](#authenticating) for each tunnel created.
