# Networking

Here's a list of ports that are exposed to the host machine. This serves as a reference to prevent setting services to ports that are already in use.

## Port Forwarding Tables

### Native Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| SSH                         | 22          | TCP      |
| Samba (SMB/CIFS)            | 445         | TCP      |

### Docker Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| Portainer                   | 8000, 9000  | TCP      |
| Fleet                       | 8080        | TCP      |

### Media Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| FreshRSS                    | 5200        | TCP      |
| Kavita                      | 5000        | TCP      |
| Jellyfin                    | 8086        | TCP      |
| EmbyStat                    | 6555        | TCP      |
| Plex                        | 32400       | TCP      |
| Tautulli                    | 8181        | TCP      |
| Synclounge                  | 8088        | TCP      |
| Ombi                        | 3579        | TCP      |
| Transmission                | 9091        | TCP      |
| Sonarr                      | 8989        | TCP      |
| Radarr                      | 7878        | TCP      |
| Jackett                     | 9117        | TCP      |
| JDownloader                 | 3129, 5800  | TCP      |

### Data Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| Ghost                       | 15800       | TCP      |
| Nextcloud                   | 9020        | TCP      |
| Gitea                       | 3000        | TCP      |
| Jenkins                     | 10800       | TCP      |

### Monitoring Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| LibreSpeed                  | 8050        | TCP      |
| Scrutiny                    | 8020        | TCP      |

### Automation Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| Custom Automation Service   | 5678        | TCP      |

### Analytics Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| Plausible                   | 10500       | TCP      |

### Other Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| RTMP Simulcast              | 1935        | TCP      |
| Webframes                   | 80          | TCP      |

### Server Monitoring Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| Prometheus                  | 9090, 9100  | TCP      |  
| Grafana **(Raspberry Pi)**  | 3000        | TCP      |

### Games

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| TeamSpeak 3 (Voice)         | 9987        | UDP      |
| TeamSpeak 3 (ServerQuery)   | 10011       | TCP      |
| TeamSpeak 3 (FileTransfer)  | 30033       | TCP      |
| Arma 3                      | 2302-2345   | TCP/UDP  |
| Assetto Corsa (Manager)     | 8772        | TCP      |
| Assetto Corsa (Server)      | 9600        | TCP/UDP  |
| Assetto Corsa (HTTP)        | 8081        | TCP      |
| Minecraft                   | 25565       | TCP      |
| Valheim                     | 2456, 2457  | UDP      |

## Deprecated Port Forwarding Tables

### Deprecated Media Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| Miniflux                    | 5190        | TCP      |

### Deprecated Data Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| Drone                       | 3080        | TCP      |
| Drone (Runner)              | 3100        | TCP      |
| Nexus                       | 10600       | TCP      |
| Nexus (Docker Registry)     | 10610       | TCP      |
| Nexus (NPM Registry)        | 10620       | TCP      |

### Deprecated Automation Services

| Service                     | Port Range  | Protocol |
|-----------------------------|-------------|----------|
| n8n                         | 5678        | TCP      |
| n8n MongoDB                 | 56787       | TCP      |

## Information

If you don't know what internal IP the server is running on, you can always type on the terminal:

```bash
ifconfig
```

## UFW

*UFW* is a friendly frontend for *iptables* that makes it a lot easier to add connection rules to your firewall. One of the many wonders of *UFW* is the fact that rules are automatically saved when set, which is not true for *iptables*.

### Installation

We'll need to install *UFW* and set it up.

```bash
sudo apt-get install ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw enable
```

The commands should be pretty self-explanatory but, just in case, these default the firewall to allow any connections going from the server to the Internet and deny any incoming connections from the Internet to the server.

### Usage

A very good command to check *UFW*'s status (if it's enabled or disabled) and see all the custom rules added and active is:

```bash
sudo ufw status
```

You can add a new rule by using:

```bash
sudo ufw allow <PORT_RANGE>/<PROTOCOL>
```

## SSH

### Installation

Since the server will run in headless mode, it would be very useful to be able to access it remotely from within (and even outside) the network. We'll use *OpenSSH*, it is generally installed with the OS but in case that it isn't, you can install it by running:

```bash
sudo apt-get install openssh-client openssh-server
```

### Setting-up

We now need to allow SSH connections through the firewall so we can access the server.

```bash
sudo ufw allow ssh
```

### Google Authentication (2FA)

Two Factor Authentication has become a must-have in terms of account security, almost all services out there have the option to add this security measure to ensure account security. Luckily, we can implement our own Two Factor Authentication to access the server through *SSH*.

#### Installation

To enable *2FA*, we'll need to install the following package:

```bash
sudo apt-get install libpam-google-authenticator
```

#### Setting-up

To set it up, simply run:

```bash
google-authenticator
```

When running this, you'll receive a *secret key* which is used to add your account manually to your phone's *2FA* application. Alternatively, you also get a nicely printed QR code on the terminal window (which you may need to resize to see fully) that you can scan with your phone. You will also get some scratch codes that you should always keep somewhere safe, just in case you lose access to your phone or something happens, you can still login to your server.

To continue, answer `y` to all the questions to set up *2FA* with the default settings.

We now need to enable *2FA* on *SSH*, to do this, edit the following file:

```bash
sudo nano /etc/ssh/sshd_config
```

Look for the following lines and edit them accordingly:

```text
UsePAM yes
ChallengeResponseAuthentication yes
```

Save and close the editor and restart the SSH service.

```bash
sudo systemctl restart ssh
```

We now need to edit the PAM rule file:

```bash
sudo nano /etc/pam.d/sshd
```

At the end of the file, add the following line:

```text
auth required pam_google_authenticator.so
```

Save and close the file.

#### Testing

In order to test that *2FA* works properly, open up a new *SSH* session without closing the previous one and try logging in, you'll be prompted for your user password and for the *2FA* code which is available on your phone.

!!! info
    When using *2FA* for *SSH*, all the users in the server will need to set-it up, otherwise they won't be able to access their accounts. In case you get locked out from one of these users, you can always login to a sudoer account (usually the admin one which is added when installing the OS) and force-login with: `sudo -iu <user>`.
