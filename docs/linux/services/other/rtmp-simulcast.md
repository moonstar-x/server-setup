# RTMP Simulcast

If you like to stream your video games to multiple platforms simultaneously, then *RTMP Simulcast* is for you. There are services out there that let you do this for free, but they have some cons:

* Depending on the service you use, your stream titles or descriptions could be used for advertising purposes, which basically means you lose complete control of what your stream description displays to your viewers.
* Since these services are used by multiple people at the same time, streaming platforms will most likely detect a huge pool of users streaming from the same IP which means that you'll be targeted by a low exposure algorithm that will, ironically, make your stream harder to find.

You can bypass these problems by streaming simultaneously to your desired platforms yourself, but keep in mind, there are some difficulties as well:

* Doing this (ideally) requires you to have a second computer (performance doesn't matter too much, you could even use a Raspberry Pi for this), you could technically achieve the same result with a virtual machine inside the streaming computer but it would represent a huge performance drop.
* Since you would be streaming to multiple platforms yourself, you would need a much higher upload bandwidth (around the stream bitrate multiplied by the number of platforms you're streaming to). For instance, my stream gets rendered at a bitrate of 6000 *kbps*, if I wanted to stream to 3 different platforms at the same time, I would need around 20000 *kbps*.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/other/rtmp-simulcast
```

## Configuration

To configure this service, we'll first create a folder called `config` inside `~/other/rtmp-simulcast` and create a file named `rtmp.conf` where the configuration for the simulcast will be located.

```bash
mkdir ~/other/rtmp-simulcast/config
nano ~/other/rtmp-simulcast/config/rtmp.conf
```

And its content should be as follows:

```text
# RTMP Stream Simulcast
rtmp {
    server {
        listen 1935;

        application live {
            live on;
            record off;
            push rtmp://ingest.server.com/application/stream_key
        }

        application local {
            live on;
            record off;
        }
    }
}
```

!!! note
    Replace `rtmp://ingest.sever.com/application/stream_key` with the actual ingest server of the platform you want to stream to. For example: `rtmp://a.rtmp.youtube.com/live2/{my_stream_key}`. You can push multiple ingest servers.

## Dockerfile

Since the service does not have an official *Docker* image, we'll create a `Dockerfile`. The content of the `Dockerfile` file is as follows:

```docker
FROM ubuntu:20.04

RUN apt-get update && apt-get install -y nginx libnginx-mod-rtmp

WORKDIR /etc/nginx
VOLUME /etc/nginx/rtmp-config

RUN printf "\n\nrtmp {\n    include /etc/nginx/rtmp-config/*.conf;\n}\n" >> nginx.conf

EXPOSE 1935
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  rtmp:
    build: .
    restart: unless-stopped
    volumes:
      - ./config:/etc/nginx/rtmp-config
    ports:
      - 1935:1935
    environment:
      - TZ=America/Guayaquil
```

Before starting, we need to build this image, do so with:

```bash
docker-compose build
```

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 1935/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.

## Streaming Settings

In your streaming software, change your streaming server to the following:

``` text
rtmp://server_local_ip/live
```

And you can place whatever you want as the stream key.

For testing purposes, you can also stream to `rtmp://server_local_ip/local` and watch it to see how it performs.

## Watching Your Stream Locally

If you wish to see how your server perceives your stream, you can use a *RTMP* player (VLC will do the trick), and open the stream at the url `rtmp://server_local_ip/local/{stream_key}`. Replace `{stream_key}` with the actual streaming key that you're currently using.
