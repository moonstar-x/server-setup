# Reddit Slideshow

[Reddit Slideshow](https://redditslideshow.com/) is a web app that can render image posts from Reddit as a slideshow. I made a little fork of the [original repo](https://github.com/ismaelpadilla/reddit-slideshow) and added a Docker image which can be accessed through [my Git server](https://code.moonstar-x.dev/public/-/packages/container/reddit-slideshow/latest).

The fork includes a little landing page with some buttons to quickly access to specific slideshows. These buttons can be configured through a `settings.json` file which will be shown later on.

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/reddit-slideshow
```

## Docker Compose

*Reddit Slideshow* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  web:
    image: code.moonstar-x.dev/public/reddit-slideshow:latest
    restart: unless-stopped
    volumes:
      - ./settings.json:/usr/share/nginx/html/settings.json
    ports:
      - 26969:8080
    environment:
      - TZ=America/Guayaquil
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

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 26969/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
