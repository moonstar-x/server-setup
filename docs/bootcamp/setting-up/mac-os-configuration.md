# macOS Configuration

!!! note
    This Mac Mini in macOS will operate mainly as a [Jenkins](../../linux/services/data/jenkins.md) agent.

Since the installation of macOS is graphically based there will not be any step by step indications of how to do it.

Instead, this page specifies the software installed on this computer.

## Applications

Here's a list of all the applications installed on the Mac Mini in macOS.

* [Chrome Remote Desktop](https://remotedesktop.google.com/access/)
* [Docker Desktop](https://desktop.docker.com/mac/main/amd64/93002/Docker.dmg)
* [Google Chrome](https://www.google.com/chrome/)
* [Homebrew](https://brew.sh/)
* [iTerm2](https://iterm2.com/)
* [Node.js](https://nodejs.org/en)
* [nvm](https://github.com/nvm-sh/nvm#installing-and-updating)
* [oh-my-zsh](https://ohmyz.sh/#install)
* [Sublime Text](https://www.sublimetext.com/)
* [Visual Studio Code](https://code.visualstudio.com/)

!!! note
    In the case of **Docker Desktop**, the link provided points to the latest download supported for macOS Catalina since newer versions require at least Big Sur.

## Homebrew Packages

Here's some commands related to *homebrew* that install some necessary packages:

```bash
brew tap adoptopenjdk/openjdk
brew install neofetch adoptopenjdk htop
```

## Jenkins Agent

Jenkins agents do not need any special software to run (other than Java). However, it is necessary to enable **SSH** connections on macOS by heading to **System Preferences > Sharing** and enabling **Remote Login**.
