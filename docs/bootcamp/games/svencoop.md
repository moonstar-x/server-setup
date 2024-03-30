# Sven Coop

Use [this script](https://github.com/moonstar-x/game-server-updater) to install the game server.

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fgame-server-updater%2Fblob%2Fmaster%2Flib%2Fservers%2Fsvencoop.py&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

## Configuration

Inside the `svencoop` folder you should create a `server.cfg` and add the commands to configure your server. You may base yourself from my own config:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Fsvencoop%2Fserver.cfg&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

## Running the Server

To run this server you should run the following command:

```text
svends.exe -console +maxplayers 8 +log on +map _server_start
```

### With a Script

You may also create a bat script to run this server easily. I like to create a `scripts` folder inside the root of the server folder and add whatever scripts I need in there.

For this case, the `scripts/start.bat` file contains the following:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2Fmoonstar-x%2Fserver-configs%2Fblob%2Fmaster%2Fsvencoop%2Fstart.bat&style=github&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>
