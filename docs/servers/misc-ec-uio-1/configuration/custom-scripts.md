# Custom Scripts

We will create some custom scripts that will help us with certain tasks. For this, we'll create the following folder:

```bash
mkdir -p /usr/local/bin
```

Then inside this folder we'll insert all the scripts that we'll add here. Make sure to make them executable with:

```bash
chmod +x <file>
```

## `custom-docker-update`

We'll use this script to manually update *docker compose* containers.

!!! info "Usage"
    Run `custom-docker-update` inside the folder where `docker-compose.yml` is located to update the container images used.

```bash
#!/bin/bash

echo "Stopping containers..."
docker compose stop

echo "Removing containers..."
docker compose rm -f

echo "Pulling images..."
docker compose pull

echo "Restarting containers..."
docker compose up -d
```

## `custom-docker-restart`

We'll use this script to completely restart *docker compose* containers, this removes the containers and restarts them.

!!! info "Usage"
    Run `custom-docker-restart` inside the folder where `docker-compose.yml` is located to update the container images used.

```bash
#!/bin/bash

docker compose rm -fs && docker compose up -d
```
