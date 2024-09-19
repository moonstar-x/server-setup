# Docker

All the services in the server will be run through *Docker*. We'll need to first install this.

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

Finally, reboot the server for the changes to apply.

## Networks

By default, *Docker* will support up to 30 networks, and since we're planning on setting up multiple services, this limit will become problematic.

In order to extend the number of networks, we will configure the daemon with the multiple subnets. For this, create the following file:

```bash
sudo nano /etc/docker/daemon.json
```

And paste the following text:

```json
{
   "default-address-pools": [
        {
            "base":"172.17.0.0/12",
            "size":16
        },
        {
            "base":"192.168.0.0/16",
            "size":20
        },
        {
            "base":"10.99.0.0/16",
            "size":24
        }
    ]
}
```

Then finally, restart the *Docker* service:

```bash
sudo service docker restart
```
