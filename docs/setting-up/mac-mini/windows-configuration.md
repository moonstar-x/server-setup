# Windows Configuration

!!! note
    This Mac Mini in Windows will operate mainly for game servers.

Since the installation of Windows is graphically based there will not be any step by step indications of how to do it.

Instead, this page specifies the software installed on this computer.

## Applications

Here's a list of all the applications installed on the Mac Mini in macOS.

* [Chrome Remote Desktop](https://remotedesktop.google.com/access/)
* [Google Chrome](https://www.google.com/chrome/)
* [Java SE](https://www.java.com/en/download/manual.jsp)
* [Python](https://www.python.org/)
* [SharpKeys](http://www.randyrants.com/category/sharpkeys/)
* [Steam](https://store.steampowered.com/about/)
* [SteamCMD](https://steamcdn-a.akamaihd.net/client/installer/steamcmd.zip)
* [Visual Studio Code](https://code.visualstudio.com/)
* [VLC](https://www.videolan.org/vlc/)
* Windows Terminal (Download from Microsoft Store)
* [Winrar](https://www.rarlab.com/)
* [ZeroTier](https://www.zerotier.com/download/)

!!! note
    **SharpKeys** is a program that allows you to remap keys in your keyboard. I use it mostly to switch the **Z** and **Y** keys on my keyboard while keeping my own keyboard layout.

## Dependencies

Since some game servers require some redistributables, here's the download links for them:

* [DirectX](https://www.microsoft.com/en-us/download/details.aspx?id=35)
* [VCRedist 2015, 2017, 2019 and 2022](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#visual-studio-2015-2017-2019-and-2022)
* [VCRedist All-in-One](https://www.techpowerup.com/download/visual-c-redistributable-runtime-package-all-in-one/)

The last download contains all the Visual C++ Redistributables except for 2022. It also comes with a handy script that installs all of them automatically.

## Scoop

Scoop is a package manager for Windows which can be installed by running the following command in an elevated powershell terminal:

```powershell
iwr -useb get.scoop.sh | iex
```

Here's some commands related to *scoop* that install some necessary packages:

```bash
scoop install git neofetch
```
