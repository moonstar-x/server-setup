# Mounting Drives

This server uses multiple USB hard drives that remain constantly connected. We need these drives to mount on system startup, for this, we'll need to set up the `fstab`.

## Getting Drives' UUIDs

In order to get the UUIDs of the drives in question, it is necessary to plug them in and reboot the server. Once this is done, execute the following:

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
sdc           
└─sdc1 ext4   43fbd170-3eff-40ec-95fc-c6cc090a5bc9
sdd           
└─sdd1 ext4   fe650c70-60fc-400a-8241-27895d38c29a
sde           
└─sde1 ext4   fedd9ed1-d4cf-4d3c-b105-7b3296f157b4
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


Disk /dev/sdc: 7.28 TiB, 8001563221504 bytes, 15628053167 sectors
Disk model: Expansion Desk  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: C581269E-50B9-4FFC-878C-4036680EF540

Device     Start         End     Sectors  Size Type
/dev/sdc1   2048 15628052479 15628050432  7.3T Microsoft basic data


Disk /dev/sdd: 3.64 TiB, 4000787027968 bytes, 7814037164 sectors
Disk model: EXTERNAL_USB    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: CB907ACB-024A-4CAC-A2F0-6A2B5B2C448D

Device     Start        End    Sectors  Size Type
/dev/sdd1   2048 7814037130 7814035083  3.6T Linux filesystem


Disk /dev/sde: 3.64 TiB, 4000752599040 bytes, 7813969920 sectors
Disk model: Elements 25A2   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: C3D4170B-C744-3D4F-BFF7-AA72B51971AE

Device     Start        End    Sectors  Size Type
/dev/sde1   2048 7813969886 7813967839  3.6T Linux filesystem
```

Notice the `Device` value for each, reference it with the output from the previous command and you'll now be able to tell the identifier of the partitions you need.

## Preparing the Folders

We'll mount the hard drives in `/media`, for this we'll create the required folders like so:

```bash
sudo mkdir /media/sata_2tb /media/usb_4tb /media/usb_4tb_2 /media/usb_8tb
```

You may also change the ownership on these folders if it causes any problem.

```bash
sudo chown 1000:1000 /media/sata_2tb /media/usb_4tb /media/usb_4tb_2 /media/usb_8tb
```

## Modifying `fstab`

!!! danger
    Proceed at your own risk, messing up this file will most probably break your computer. You can still fix it by entering safe mode and logging in as `root` to rollback.

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
UUID=418987da-2351-11e9-aec8-b8975ad7798b /media/sata_2tb ext4 defaults 0 0
UUID=43fbd170-3eff-40ec-95fc-c6cc090a5bc9 /media/usb_8tb ext4 defaults 0 0
UUID=fedd9ed1-d4cf-4d3c-b105-7b3296f157b4 /media/usb_4tb ext4 defaults 0 0
UUID=fe650c70-60fc-400a-8241-27895d38c29a /media/usb_4tb_2 ext4 defaults 0 0
```

Finally, reboot the server. The hard drives should now be automatically mounted.
