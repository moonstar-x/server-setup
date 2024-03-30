# Reddit Slideshow

[Reddit Slideshow](https://redditslideshow.com/) is a web app that can render image posts from Reddit as a slideshow.

I made a little fork of the [original repo](https://github.com/ismaelpadilla/reddit-slideshow) and added a Docker image which can be publicly used [code.moonstar-x.dev/public/reddit-slideshow](https://code.moonstar-x.dev/public/-/packages/container/reddit-slideshow/latest).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/reddit-slideshow
```

Create a `settings.json` file and add something like this:

```json
{
  "slideshows": [
    "funny",
    ["memes", "pics"]
  ]
}
```

You can add the subreddits you want as a string for single subreddits or as an array for multireddit.

## Docker Compose

*Reddit Slideshow* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: code.moonstar-x.dev/public/reddit-slideshow:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - reddit_slideshow
    volumes:
      - ./settings.json:/usr/share/nginx/html/settings.json
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.reddit_slideshow.rule: Host(`slideshow.home.arpa`)
      traefik.http.routers.reddit_slideshow.entrypoints: local
      traefik.http.routers.reddit_slideshow.service: reddit_slideshow@docker
      traefik.http.services.reddit_slideshow.loadbalancer.server.port: 8080

networks:
  proxy_external:
    external: true
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
