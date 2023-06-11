# OS Installation

The server will be running [Ubuntu Server 20.04 LTS 64-bit](https://ubuntu.com/download/server) in headless mode, meaning that no DE will be used. The installation is quick and assisted by its own install wizard. When asked what snapshot (initial server configuration) should be installed, simply choose **none** or **default**.

As a general rule of thumb, after installing the OS it is recommended to update the sources ad packages:

```bash
sudo apt-get update && sudo apt-get upgrade
```

## Configuring Date and Time

By default, the OS will be installed with **GMT+0** as the timezone. We'll change this to conform with our real timezone which is **GMT-5**.

```bash
sudo timedatectl set-timezone America/Guayaquil
```
