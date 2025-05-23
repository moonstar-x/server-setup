# Installation

The server will be running [Ubuntu Server 24.04 LTS 64-bit](https://ubuntu.com/download/server) in headless mode, meaning that no DE or WM will be used. The installation is quick and assisted by its own installation wizard. For better performance, you should install the minimal version which does not include any graphical interface other than the command line. Additionally, no packages were installed from the installation wizard.

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
sudo apt-get install openssh-server openssh-client net-tools neofetch nload progress nano iputils-ping htop git speedtest-cli
```

## Firewall

*UFW* is a friendly frontend for *iptables* that makes it a lot easier to add connection rules to your firewall.

We'll need to install *UFW* and set it up.

```bash
sudo apt-get install ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw enable
```

The commands should be pretty self-explanatory, these default the firewall to allow any connections going from the server to the Internet and deny any incoming connections from the Internet to the server.

Make sure to also enable the `ssh` ports, otherwise you may get locked out of your system remotely and may need to physically configure the firewall to allow SSH connections.

### Usage

A very good command to check *UFW*'s status (if it's enabled or disabled) and see all the custom rules added and active is:

```bash
sudo ufw status
```

You can add a new rule by using:

```bash
sudo ufw allow <PORT_RANGE>/<PROTOCOL>
```

## Git

By default, `git` does not have a credential store configured, so make sure you run the following command to allow git operations in protected repos:

```bash
git config --global credential.helper store
```
