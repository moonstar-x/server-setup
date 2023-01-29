# Nexus

!!! warning "Deprecation Warning"
    This service has been deprecated in favor of [Gitea](../../services/data/gitea.md)'s [Package Registry](https://docs.gitea.io/en-us/packages/overview/).

[Nexus](https://www.sonatype.com/products/repository-oss-download) is an artifact repository that supports various types of formats, including Docker images and NPM packages. This service has an official image available on [Docker Hub](https://hub.docker.com/r/sonatype/nexus3) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/nexus
```

## Docker Compose

*Nexus* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  nexus:
    image: sonatype/nexus3:latest
    restart: unless-stopped
    user: 1000:1000
    volumes:
      - ./data:/nexus-data
    ports:
      - 10600:8081
      - 10610:10610
      - 10620:10620
    environment:
      - TZ=America/Guayaquil
```

## Post-Installation

To avoid permission issues, create the `data` folder that will be used as a volume:

```bash
mkdir data
```

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 10600/tcp
sudo ufw allow 10610/tcp
sudo ufw allow 10620/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.

## Configuration

Follow the following guidelines to configure the registry dashboard.

### Creating an Admin Account

Inside the stack's folder, run the following command:

```bash
cat data/admin.password
```

This will print a random string that will be the first administrator's password.

Now, visit the registry's dashboard at:

```text
http://localhost:10600
```

And login with the `admin` username and the password recovered previously.

### Setting Up Blob Storage

It is preferable to have a blob storage for each repository. Since we'll have NPM and Docker registries, we'll create
a blob storage for each one of them.

Create 2 blob storages with the `docker` and `npm` names and remove the `default` blob storage.

### Setting Up Repositories

Remove the base repositories included. Now, create the following repositories:

1. Docker Registry
    * Type: `Docker (hosted)`
    * Name: `docker`
    * HTTP Connector: `10610`
    * Blob Store: `docker`
    * Deployment Policy: `Allow redeploy`

2. NPM Registry
    * Type: `NPM (hosted)`
    * Name: `npm`
    * HTTP Connector: `10620`
    * Blob Store: `npm`
    * Deployment Policy: `Allow redeploy`

### Create Some Roles

We'll create some roles to create limited users to use with the registry.

Create the following roles:

1. Docker Role
    * Role ID: `custom-docker`
    * Role Name: `custom-docker`
    * Role Description: `Allow Docker operations.`
    * Privileges: `nx-repository-view-docker-*-*`

2. NPM Role
    * Role ID: `custom-npm`
    * Role Name: `custom-npm`
    * Role Description: `Allow NPM operations.`
    * Privileges: `nx-repository-view-npm-*-*`

### Create Some users

It is recommended to create a new administrator user and remove the default one.

Now, create an user that you'll use for each of the registries created and give it only access to the corresponding role that was created
previously for each of the registries.

For example, create a user `user-docker` that has only access to the `custom-docker` role. Now you can use this user to login
with Docker with the command:

```bash
docker login
```

!!!note
    As a last recommendation, create additional users with limited roles to use with any CI system.
