# Arma 3

Use [this script](https://github.com/moonstar-x/game-server-updater) to install the game server.

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fgame-server-updater%2Fblob%2Fmaster%2Flib%2Fservers%2Farma3.py&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

This script is a bit different from the rest because it has a post run function that will automatically create symlinks for the mod folders and will update the mod keys inside the `keys` folder of the server.

Running this script is enough to get the server ready with mods installed.

## Configuration

Inside the server folder you should create a `scripts` folder and then use subfolders for each server configuration you need. In my case, one of my subfolders is named `horror_cup`.  Inside this subfolder, you should have a `server.cfg` file. You can base yourself off the following:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Farma3%2Fhorror_cup%2Fserver.cfg&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

## Running the Server

To run this server you should create a script inside the subfolder of the `scripts` folder. In my case, one of my subfolders is named `horror_cup`. Inside this subfolder, you should have a `start.bat` file. You can base yourself off the following:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Farma3%2Fhorror_cup%2Fstart.bat&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

The `MODLIST` variable contains the names of the symlinked folders for the mods to load on the server. This should be a string with the names of the mod folders separated by a `;` character. For unix users, the `;` character should be escaped (`\;`).

## Running the Server with Headless Client

If you wish to run the server with a headless client alongside the server on the same machine, you can base yourself of the following `start.bat` file.

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Farma3%2Fantistasi_rhs_cup_maps%2Fstart.bat&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>
