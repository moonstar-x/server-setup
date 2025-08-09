# Installation

Flash your SD Card with the OS of your choice, in this case, we'll be using **Raspberry Pi OS Lite**. Head over to the [Operating Systems](https://www.raspberrypi.com/software/operating-systems/) page for more information. You can choose **Raspberry Pi OS Desktop**, however the *Lite* version will contain much less clutter that will be unnecessary.

## Post-Installation

As a general rule of thumb, after installing the OS it is recommended to update the sources and packages:

```bash
sudo apt-get update && sudo apt-get upgrade
```

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

And pick your own timezone.

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

### Enable Autologin

We need this machine to automatically log with the admin user, for this choose the following options:

```text
System Options > Auto Login > Enable
```

The user will be automatically logged in on system startup.

## Packages

Here's a list of packages I like to have on the server.

```bash
sudo apt-get install tmux xterm git neofetch
```

## Installing `gotop`

`gotop` is a pretty performance monitor that display a nice UI on the terminal. To install this, run the following commands:

```bash
git clone --depth 1 https://github.com/cjbassi/gotop /tmp/gotop
/tmp/gotop/scripts/download.sh
sudo mv gotop /usr/local/bin
```

## Generate SSH Key

This server in particular will SSH to other machines to monitor them with `gotop`. For this we'll create an SSH key with the following command:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Then, copy the contents of the `~/.ssh/ed25519.pub` file into the destination server's `~/.ssh/authorized_keys` file.

## Installing a WM

For this server we will need a GUI. We'll install a simple WM to avoid overwhelming the Raspberry Pi with heavy usage that might hinder our experience. For this, we'll use [Awesome WM](https://awesomewm.org/).

First, install *X* and *Awesome*:

```bash
sudo apt-get install xinit awesome 
```

And start it,

```bash
startx
```

We'll install a theme to the WM so it does not look too bare-bones. We'll use [awesome-copycats](https://github.com/lcpz/awesome-copycats) for this. Install this with:

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

```bash
if [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
  exec startx
fi
```

## Customizing Terminal

Create a `.Xresources` file with:

```bash
nano ~/.Xresources
```

And paste the following content:

```text
*foreground:   #c5c8c6
*background:   #1d1f21
*cursorColor:  #aeafad
*color0:       #000000
*color1:       #912226
*color2:       #778900
*color3:       #ae7b00
*color4:       #1d2594
*color5:       #682a9b
*color6:       #2b6651
*color7:       #929593
*color8:       #666666
*color9:       #cc6666
*color10:      #b5bd68
*color11:      #f0c674
*color12:      #81a2be
*color13:      #b294bb
*color14:      #8abeb7
*color15:      #ecebec
```

Now the terminal should have pretty colors.

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
