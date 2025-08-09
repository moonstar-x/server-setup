# Docker

Some services in the server will be run through *Docker*. We'll need to install it first.

## Installation

To install *Docker*, simply run the following commands:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

Once it's done, you can remove the downloaded script:

```bash
rm get-docker.sh
```

## Permissions

We'll add the required permissions for our user into the `docker` group.

```bash
sudo groupadd docker
sudo gpasswd -a $USER docker
```
