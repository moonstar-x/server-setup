# Arma 3 (Native)

[Arma 3](https://store.steampowered.com/agecheck/app/107410/) is an excellent military simulation game. I haven't been able to find any Docker image for this server that is maintained and does not require to disable Steam Guard from your account. For this reason, this server will be installed natively.

## User Creation

For this server, we'll create a separate user where all the server data will be located.

```bash
sudo adduser --disabled-login arma
```

!!! note
    For future reference, you may switch to this user by running `sudo -iu arma`.

## Dependencies

The **Arma 3** server requires some dependencies that we'll install:

```bash
sudo apt-get install lib32gcc1 lib32stdc++6
```

## Installing SteamCMD

SteamCMD is a headless Steam client that is generally used to download dedicated servers. We'll need this in order to install the **Arma 3** server.

First, let's switch to the `arma` user:

```bash
sudo -iu arma
```

We'll also create a folder named `steamcmd`:

```bash
mkdir steamcmd && cd steamcmd
```

Then, download and extract SteamCMD:

```bash
wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar zxvf steamcmd_linux.tar.gz && rm steamcmd_linux.tar.gz
```

Once this is done, you may execute `steamcmd` for the first time so that all its data can be downloaded.

```bash
./steamcmd
```

## Installing Arma 3 Server

In order to install the server, start up `steamcmd` then run the following commands:

```text
login <username>
force_install_dir /home/arma/arma3
app_update 233780
```

!!! note
    Replace `<username>` with your Steam username. Once you run the login command, `steamcmd` will ask for your password and your Steam Guard code.

## Mods

### Setting Up Case Insensitive On Purpose (ciopfs)

On a Linux based installation, **Arma 3** modding becomes a bit complicated because of the Unix filesystem's case sensitivity, which renders mods with capital letters in their filenames unusable. Luckily, we can fix this with a nice package called `ciopfs`.

As a sudoer account (not `arma`), run:

```bash
sudo apt-get install ciopfs
```

Switch back to the `arma` user. We'll create the folders for the mods and run `ciopfs` on them:

```bash
mkdir arma3mods arma3mods_ciopfs
ciopfs arma3mods arma3mods_ciopfs
```

What this does is mount the commands of `arma3mods` in `arma3mods_ciopfs` in a case insensitive fashion. This means that new content should be added to the `arma3mods_ciopfs` folder, this will make the `arma3mods` folder contain the corresponding data with case insensitivity.

The issue with this is that this change will be undone on system restart. For this reason, we'll create a service to run this on system start.

As a sudoer account, run:

```bash
sudo nano /etc/systemd/system/arma3ciopfs.service
```

And paste the following:

```text
Description=Arma 3 ciopfs mount.

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
User=arma
ExecStart=/usr/bin/ciopfs /home/arma/arma3mods /home/arma/arma3mods_ciopfs
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

Finally, enable the service:

```bash
sudo systemctl enable arma3ciopfs.service
sudo systemctl start arma3ciopfs.service
```

### Installing Mods

With the `arma` user, run `steamcmd` and run the following commands:

```text
login <username>
force_install_dir /home/arma/arma3mods_ciopfs
workshop_download_item 107410 <mod_id>
```

!!! note
    You may need to re-run the last command a second time to make sure the mod does get downloaded.

Once installed, we'll need to create symlinks from the mod folders to the **Arma 3** server directory. For this, run the following command:

```bash
ln -s ~/arma3mods/steamapps/workshop/content/107410/<mod_id> ~/arma3/@<mod_name>
```

As an example, for the **CBA_A3** mod:

```bash
ln -s ~/arma3mods/steamapps/workshop/content/107410/450814997 ~/arma3/@CBA_A3
```

Finally, we'll also need to copy the keys for each mod into the `keys` folder inside the server directory. Mod keys will be in `~/arma3mods/steamapps/workshop/content/107410/<mod_id>/keys` (sometimes the last folder will be `key` rather than `keys`). These keys will have a `.bikey` extension and will need to copied to `~/arma3/keys`.

!!! note
    Every time a mod gets updated, you need to replace the old keys with the new ones.

### Mods List

Here's a small list of the mods that I currently use:

| Workshop ID | Addon Name                              | Symlink Name              |
|-------------|-----------------------------------------|--------------------------|
| 463939057   | ACE                                     | @ACE                     |
| 450814997   | CBA_A3                                  | @CBA_A3                  |
| 621650475   | CUP ACE3 Compatibility Addon - Vehicles | @CUP_ACE_compat_vehicles |
| 549676314   | CUP ACE3 Compatibility Addon - Weapons  | @CUP_ACE_compat_weapons  |
| 583496184   | CUP Terrains - Core                     | @CUP_Core                |
| 853743366   | CUP Terrains - CWA                      | @CUP_CWA                 |
| 583544987   | CUP Terrains - Maps                     | @CUP_Maps                |
| 497661914   | CUP Units                               | @CUP_Units               |
| 541888371   | CUP Vehicles                            | @CUP_Vehicles            |
| 497660133   | CUP Weapons                             | @CUP_Weapons             |
| 333310405   | Enhanced Movement                       | @Enhanced_Movement       |
| 843425103   | RHSAFRF                                 | @RHSAFRF                 |
| 843593391   | RHSGREF                                 | @RHSGREF                 |
| 843632231   | RHSSAF                                  | @RHSSAF                  |
| 843577117   | RHSUSAF                                 | @RHSUSAF                 |
| 498740884   | ShackTac User Interface                 | @ShackTac_UI             |
| 620019431   | TaskForce Arrowhead Radio               | @task_force_radio        |
| 501966277   | Zombies and Demons                      | @Zombies_and_Demons      |

### Infinite Loading Screen Bug

Unfortunately, the Linux Arma 3 server can be a bit buggy, this is the case for servers that run certain mod combinations (in this case CUP and RHS) that will make clients get stuck on an infinite loading screen when joining the server, the server console will not display any error messages and will in fact freeze the process. It seems the only way to fix this issue is to disable BattlEye from the server configs. In the server config, change the following:

``` text
BattlEye = 0;
```

This will in fact turn off BattlEye on your server which will make it less secure against hackers, sadly it is the only fix for this weird problem.

## Configuration

### Server Profile

**Arma 3** uses profile files to define difficulty settings and vars. We'll need to first create the corresponding folder, run:

```bash
mkdir -p ~/".local/share/Arma 3" && mkdir -p ~/".local/share/Arma 3 - Other Profiles"
cd ~/.local/share/Arma\ 3\ -\ Other\ Profiles/ && mkdir server && cd server
```

Then, inside this folder, create a file named `server.Arma3Profile` with:

```bash
nano server.Arma3Profile
```

Inside, paste the following:

```text
version=1;
blood=1;
singleVoice=0;
gamma=1;
brightness=1;
maxSamplesPlayed=96;
activeKeys[]=
{
  "BIS_Dynamic Recon Ops - Chernarus Winter.Chernarus_winter_done",
  "BIS_creeping death.altis_done",
  "BIS_co10_Escape_CUP_RU_vs_BAF.Chernarus_done",
  "BIS_co10_Escape_RHS_RU_vs_US.Chernarus_done",
  "BIS_COOP1-6%20A%20Cabin%20in%20the%20Hills.chernarus_done",
  "BIS_co10_Escape_RHS_RU_vs_US.Takistan_done"
};
playedKeys[]={};
difficulty="Custom";
class DifficultyPresets
{
  class CustomDifficulty
  {
    displayName="Custom";
    optionDescription="Custom Difficulty";
    levelAI="AILevelMedium";
    class Options
    {
      reducedDamage=0;
      groupIndicators=2;
      friendlyTags=2;
      enemyTags=0;
      detectedMines=0;
      commands=2;
      waypoints=2;
      tacticalPing=1;
      weaponInfo=1;
      stanceIndicator=1;
      staminaBar=0;
      weaponCrosshair=0;
      visionAid=0;
      thirdPersonView=0;
      cameraShake=1;
      scoreTable=1;
      deathMessages=1;
      vonID=1;
      mapContent=0;
      autoReport=0;
      multipleSaves=1;
    };
    aiLevelPreset=1;
  };
  class CustomAILevel
  {
    skillAI=0.5;
    precisionAI=0.5;
  };
};
sceneComplexity=400000;
shadowZDistance=100;
viewDistance=4000;
preferredObjectViewDistance=1000;
terrainGrid=25;
vonRecThreshold=0.029999999;
soundEnableEAX=1;
soundEnableHW=0;
volumeCD=10;
volumeFX=10;
volumeSpeech=10;
volumeVoN=10;
volumeMapDucking=1;
```

### Server Config

Now, we'll go to the folder where we installed *Arma 3* and create a config file with whatever name you wish:

``` text
cd ~/arma3
nano modded.cfg
```

You can paste the following config and change it or use whichever you want:

``` text
steamPort = 2303;
steamQueryPort = 2304;

hostname = "SERVER NAME";
password = "";
passwordAdmin = "SECRET";
maxPlayers = 32;
persistent = 0;

disableVoN = 0;
vonCodecQuality = 20;

voteMissionPlayers = 1;
voteThreshold = 0.66;
allowedVoteCmds[] =
{
    {"admin", false, false},
    {"kick", false, true, 0.51},
    {"missions", false, true},
    {"mission", false, false},
    {"restart", false, true},
    {"reassign", false, false}
};

timeStampFormat = "short";
logFile = "server_console.log";

BattlEye = 1;
VerifySignatures = 2;
kickDuplicate = 1;
allowedFilePatching = 1;

allowedLoadFileExtensions[] = {"hpp","sqs","sqf","fsm","cpp","paa","txt","xml","inc","ext","sqm","ods","fxy","lip","csv","kb","bik","bikb","html","htm","biedi"};
allowedPreprocessFileExtensions[] = {"hpp","sqs","sqf","fsm","cpp","paa","txt","xml","inc","ext","sqm","ods","fxy","lip","csv","kb","bik","bikb","html","htm","biedi"};
allowedHTMLLoadExtensions[] = {"htm","html","php","xml","txt"};

onUserConnected = "";
onUserDisconnected = "";
doubleIdDetected = "";
onUnsignedData = "kick (_this select 0)";
onHackeddData = "kick (_this select 0)";

headlessClients[] = {"127.0.0.1"};
localClient[] = {"127.0.0.1"};
```

As a side note, if you need to run servers with multiple configurations, you can always create more config files and use them at the server start.

## Custom Missions

In order to run custom missions on your server, you'll need to add the *.pbo* files inside the *mpmissions* folder inside the *Arma 3* server folder. You can get the *.pbo* files directly from Steam Workshop if you download the missions through this [Workshop Downloader](https://steamworkshopdownloader.io/) or from [Armaholic](http://www.armaholic.com/list.php?c=arma3_files_scenarios_mpmissions).

## Firewall Rules

Again, we'll need to add the following firewall rules:

``` text
sudo ufw allow 2302:2345/tcp
sudo ufw allow 2302:2345/udp
```

## Running the Server

Now, to run the game, we'll create a small script so we can easily save the addons that we want to run and the configs that we want to load. Inside the *Arma 3* server directory, create the script file (the filename with whatever you want):

``` text
nano start_<name>.sh
```

Inside the editor, paste the following:

``` bash
#!/bin/bash
./arma3server -name=<server_name> -config=<config_file> -mod=@mod1\;@mod2\;@mod3...
```

!!!note
    *server_name* refers to the server name that you used for the *Arma 3 Profile* that is located in **~/.local/share/Arma 3 - Other Profiles**, the *config_file* refers to the *Arma 3 Config* that is inside the *Arma 3* directory, replace them with your own settings.

An example for this script that runs ACE and CBA_A3.

``` bash
#!/bin/bash
./arma3server -name=server -config=modded.cfg -mod=@CBA_A3\;@ACE
```

As per always, the script must be made executable:

``` text
chmod +x <script_name>.sh
```

## Headless Client

A headless client is an instance of the game with no interface. The idea of having a headless client on a server is to reduce the load on the server process by taking control of the AI processing. This will (ideally) make the server run better and the AI more responsive since their processing will be done somewhere else, their reaction times will be much quicker.

Setting up a headless client is a trivial task. The first thing that should be done is modify the server config (`.cfg` file) to allow our headless client.

In `server.cfg` (or whichever it is the name of your `.cfg` file), add or edit the following lines with the IP address of your headless client:

```text
headlessClients[] = {"127.0.0.1"};
localClient[] = {"127.0.0.1"};
```

!!! note
    In this case, our headless client will be running in the same machine as the server, which means that `127.0.0.1` will be used. You can add as many of these as you want.

Once the server config allows the headless client, we'll create a run script to start the headless client.

``` bash
#!/bin/bash
./arma3server -client -connect=SERVER_IP -password=SERVER_PASSWORD -mod=@MOD1\;@MOD2...
```

!!! note
    You can omit the `password` directive if the server you're connecting to has no password. You can also omit the `mod` directive if the server has no mods.

Finally, make the script executable:

``` text
chmod +x start_hc.sh
```

Now you're ready. Start the server and then the headless client, which will connect directly and automatically offload the AI processing (if the mission implements headless clients properly).

### Start Server and Headless Client Script

If you wish to run them both directly, you can create the following script:

``` bash
#!/bin/bash
echo "Launching server..."
screen -dmS arma3-server /bin/bash -c "./start_antistasi.sh"

echo "Launching Headless Client..."
screen -dmS arma3-hc /bin/bash -c "./hc_antistasi.sh"
```

!!! note
    Replace `"./start_antistasi.sh"` with the name of the server start script and `"./hc_antistasi.sh"` with the name of the headless client start script.

This will start both processes in two different `screen` windows. You can switch to them by using:

``` text
screen -rd arma3-server
screen -rd arma3-hc
```
