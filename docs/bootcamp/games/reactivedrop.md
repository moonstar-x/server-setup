# Alien Swarm Reactive Drop

Use [this script](https://github.com/moonstar-x/game-server-updater) to install the game server.

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fgame-server-updater%2Fblob%2Fmaster%2Flib%2Fservers%2Freactivedrop.py&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

## Sourcemod

In order to add plugins on your server, you'll need [Sourcemod](https://www.sourcemod.net/downloads.php?branch=stable) and [Metamod](https://www.sourcemm.net/downloads.php?branch=stable), click on the mentioned links to download the necessary files.

These archives will include folders named `addons` and `cfg` which should be placed inside the `reactivedrop` folder of the server.

### Updating Admins

Some commands will require you to be an admin. First, you should get your steam user's **STEAM32 ID** which can be found [here](https://steamidfinder.com/). Once you have this, append the following line inside the `reactivedrop/addons/sourcemod/configs/admins_simple.ini` file:

```text
STEAM_0:1:23456789 99:z
```

!!! info
    Replace `STEAM_0:1:23456789` with your actual Steam ID.

## Plugins

Currently this server does not have any plugins set up.

## Configuration

Inside the `reactivedrop/cfg` folder you should create a `server.cfg` and add the commands to configure your server. You may base yourself from my own config:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Freactivedrop%2Fserver.cfg&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

Additionally to this file, you need a `workshop.cfg` file inside `reactivedrop/cfg` and include commands to download workshop content into the server (not necessary though).

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Freactivedrop%2Fworkshop.cfg&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

## Running the Server

To run this server you should run the following command:

```text
srcds.exe -game reactivedrop -console -maxplayers 8 +map lobby
```

### With a Script

You may also create a bat script to run this server easily. I like to create a `scripts` folder inside the root of the server folder and add whatever scripts I need in there.

For this case, the `scripts/start.bat` file contains the following:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Freactivedrop%2Fstart.bat&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>
