# Docker

A good portion of our services will be run through *Docker*. We'll need to first install this.

## Installation

First we'll need to install some dependencies:

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```

We'll then need to add the *Docker* repository:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Finally, we'll update the repositories and install *Docker*:

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## Permissions

We'll add the required permissions for our user into the `docker` group.

```bash
sudo groupadd docker
sudo gpasswd -a $USER docker
```

Finally, reboot the server for the changes to apply.

## Docker Compose

We'll be using *Docker Compose* to run our containers, to install it run the following:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```
