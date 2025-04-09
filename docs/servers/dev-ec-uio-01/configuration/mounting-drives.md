# Mounting Drives

This server has at least one USB hard drive that remains constantly connected. We need this drive to mount on system startup, for this, we'll need to set up the `fstab`.

## Getting Drive UUIDs

In order to get the UUIDs of the drive in question, it is necessary to plug them in. Once this is done, execute the following:

```bash
lsblk -o NAME,FSTYPE,UUID
```

This will give an output similar to the following:

```text
NAME   FSTYPE UUID
sda           
├─sda1 vfat   7811-FFBE
└─sda2 ext4   3c98de9f-dc43-4e8d-85aa-767475ea957b
sdb    ext4   418987da-2351-11e9-aec8-b8975ad7798b
```

If you cannot tell which drive is which, you can also run the following:

```bash
sudo fdisk -l
```

Which will give out an output similar to the following:

```text
Disk /dev/sdb: 1.82 TiB, 2000398934016 bytes, 3907029168 sectors
Disk model: Hitachi HUA72302
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 223.57 GiB, 240057409536 bytes, 468862128 sectors
Disk model: CT240BX500SSD1  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 18C45280-7C95-4D8C-8033-81277842D6FF

Device       Start       End   Sectors   Size Type
/dev/sda1     2048   2203647   2201600     1G EFI System
/dev/sda2  2203648 468858879 466655232 222.5G Linux filesystem
```

Notice the `Device` value for each, reference it with the output from the previous command, and you'll now be able to tell the identifier of the partitions you need.

## Preparing the Folders

We'll mount the hard drives in `/media`, for this we'll create the folders like so:

```bash
sudo mkdir /media/my_drive
```

You may also change the ownership on these folders if it causes any problem.

```bash
sudo chown -R 1000:1000 /media/my_drive
```

## Modifying `fstab`

!!! danger
    Make sure to have physical access to the server in case editing this file breaks your installation. If your file is not correct, the system will boot in safe-mode with SSH disabled.

We'll modify the `/etc/fstab` file.

```bash
sudo nano /etc/fstab
```

And we'll add a new line for each hard drive with the following structure:

```text
UUID=$UUID $DIR $FORMAT defaults 0 0
```

In our case, we'll add the following lines:

```text
UUID=418987da-2351-11e9-aec8-b8975ad7798b /media/my_drive ext4 defaults 0 0
```

Finally, reboot the server. The drive should now be automatically mounted.
