# Installation

Since the VPS comes with the OS already installed, nothing else must be done.

## Post-Installation

As a general rule of thumb, after installing the OS it is recommended to update the sources and packages:

```bash
sudo apt-get update && sudo apt-get upgrade
```

## Configuring Date and Time

By default, the OS will be installed with **GMT+0** as the timezone. We'll change this to conform with our real timezone which is **GMT-5**.

```bash
sudo timedatectl set-timezone America/Guayaquil
```

## Packages

Here's a list of packages I like to have on the server.

```bash
sudo apt-get install openssh-server openssh-client net-tools neofetch nload progress nano htop git
```

## Firewall

[Hetzner](https://www.hetzner.com/) comes with a free Firewall service that I recommend you use to only allow certain ports on certain interfaces.

I won't go much into detail because the rules depend heavily on your use case, though I recommend you be as restrictive as possible when defining ports and interfaces.

## Git

By default, `git` does not have a credential store configured, so make sure you run the following command to allow git operations in protected repos:

```bash
git config --global credential.helper store
```
