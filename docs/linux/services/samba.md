# Samba (Native)

*Samba* lets your Linux based server share files and folders on a Windows File Sharing Workgroup using the same protocol (SMB/CIFS), this is pretty useful when you need to share files between computers on your network. This also helps to allow your files to be accessed through the Internet (although it should only be done through a VPN for security purposes).

## Installation

To install *Samba*:

```bash
sudo apt-get install samba
```

## Configuring

We'll make a folder on our 2TB SATA hard drive directory, this will serve as the share folder which will be accessible through *SMB*.

```bash
mkdir /media/sata_2tb/sambashare
```

We now need to add this folder entry to the configuration. Open up *Samba*'s config file.

```bash
sudo nano /etc/samba/smb.conf
```

Now we'll add the folder we created alongside other directories to the *Samba* share (note that you should replace these directories corresponding to your needs). You can add as many entries as you need as long as they're well formatted.

```text
[sambashare]
    comment = Samba shared folder
    path = /media/sata_2tb/sambashare
    read only = no
    browsable = yes
    writeable = yes
```

Now we restart the *Samba* service to apply the changes:

```bash
sudo service smbd restart
```

Before trying to access the share with another computer, we'll need to add a password to the *Samba* user. Use the next command and replace **$USER** with your username.

```bash
sudo smbpasswd -a $USER
```

Now, as a final step, add the firewall rule to allow *SMB* connections.

```bash
sudo ufw allow 445/tcp
```

## Issues with Permissions on Windows

In case you try to access a shared folder that doesn't have the corresponding permissions to allow the Windows user to manage said folder, you can add the following directives to each shared folder block in the config file:

```text
create umask = 0777
directory umask = 0777
```

## Shared Folders that Contain Symlinks

If one of your shared folders has symlinks in them and you need to share them too, add the following directives to the shared folder block in the config file:

```text
follow symlinks = yes
wide links = yes
```

## Browsing a Shared Folder as a Specified User

If you want your shared folder to be accessed as a certain user, add the following directive to the shared folder block in the config file:

```text
force user = $USER
```

!!! note
    Replace `$USER` with the UNIX username required.
