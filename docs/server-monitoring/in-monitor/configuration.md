# Configuration

!!! info
    This page depends on the monitoring device that will be used. In this case, a Raspberry Pi 3B+ will be used. If you do not have one you might need to skip this guide and go straight to the [Grafana](./grafana.md) page.

## OS Installation

Flash your SD Card with the OS of your choice, in this case, we'll be using **Raspberry Pi OS Lite**. Head over to the [Operating Systems](https://www.raspberrypi.com/software/operating-systems/) page for more information. You can choose **Raspberry Pi OS Desktop**, however the *Lite* version will contain much less clutter that will be unnecessary.

## Configuring

Before continuing we must change some system information using the `raspi-config` utility.

!!! note
    You will need to reboot your device after completing all these changes in order for them to have effect.

### Change the Hostname

The hostname is the name of the machine, by default it is set to `pi` which might not be descriptive enough. In order to change it, open the configurator utility:

```bash
sudo raspi-config
```

And choose the following options:

```text
System Options > Hostname
```

And insert the hostname you wish to use.

### Disable Screen Blanking

Since this will be an always-on monitoring dashboard, we must disable the screen from turning off. Inside the `raspi-config` utility, select the options:

```text
Display Options > Screen Blanking
```

And disable it.

### Change the Timezone

Your Raspberry Pi might have the incorrect timezone set, it is very important to have the correct time in your device to avoid problems with connectivity. Inside the `raspi-config` utility, select the options:

```text
Localisation Options > Timezone
```

And pick the your own timezone.

### Connect to Wi-Fi

In the case that using Ethernet is not possible, Wi-Fi will be necessary. Since we do not have a GUI yet, we need to connect to the Internet through the `raspi-config`. Inside the utility, select the options:

```text
System Options > Wireless LAN
```

Insert your Wi-Fi's SSID and password to connect.

### Enable SSH

Remote control will be necessary to avoid having to use the machine directly. For this, SSH should be enabled. In the same `raspi-config` utility, select the options:

```text
Interface Options > SSH > Enable
```

SSH should now be enabled.

## Installing Software

Before installing anything, let's update the repositories and installed packages:

```bash
sudo apt-get update && sudo apt-get upgrade
```

And installed the required programs:

```bash
sudo apt-get install xterm firefox-esr git 
```

## Installing a WM

In order to access the [Grafana](./grafana.md) site, a GUI is indeed necessary. We'll install a simple WM to avoid overwhelming the Raspberry Pi with heavy usage that might hinder our experience. For this, we'll use [Awesome WM](https://awesomewm.org/).

First, install *X* and *Awesome*:

```bash
sudo apt-get install xinit awesome 
```

And start it,

```bash
xinit && awesome
```

We'll instal a theme to the WM so it doesn't look too bare-bones. We'll use [awesome-copycats](https://github.com/lcpz/awesome-copycats) for this. Install this with:

```bash
git clone --recurse-submodules --remote-submodules --depth 1 -j 2 https://github.com/lcpz/awesome-copycats.git
mkdir -p ~/.config/awesome
mv -bv awesome-copycats/{*,.[^.]*} ~/.config/awesome; rm -rf awesome-copycats
cd ~/.config/awesome
cp rc.lua.template rc.lua
```

Finally, restart awesome by right-clicking anywhere on the desktop and selecting the *Restart* option.

In order for the Raspberry Pi to automatically open *Awesome* on system boot, edit the following file:

```bash
sudo nano /etc/profile
```

And add the following lines:

```text
if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
  exec startx
fi
```

## Installing Docker

Finally, we'll need *Docker* to host the [Grafana](./grafana.md) service. To install it, run:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

And finally, reboot your machine with:

```bash
sudo reboot
```

You should now be able to execute *Docker* without issues. Make sure your user has the required permissions with:

```bash
docker info
```
