# Mounting Drives

In this server there will be a 2TB SATA drive, and 2 USB 3.0 drives of 1TB and 4TB respectively. We need these drives to mount on system startup, for this, we'll need to set up the `fstab`.

## Getting Drives' UUIDs

In order to get the UUIDs of the drives in question, it is necessary to plug them in and reboot the server. Once this is done, execute the following:

```bash
lsblk -o NAME,FSTYPE,UUID
```

This will give an output like the following:

```text
NAME   FSTYPE   UUID
loop0  squashfs
loop1  squashfs
loop2  squashfs
loop3  squashfs
loop4  squashfs
loop5  squashfs
loop6  squashfs
loop7  squashfs
sda
└─sda1 ext4     e8f133c2-af07-4a7a-a42f-21f857a9a06f
sdb
├─sdb1 vfat     A12E-F096
└─sdb2 ext4     c1aafa14-c857-4d3e-8ae1-00c8cd462810
sdc    ext4     418987da-2351-11e9-aec8-b8975ad7798b
sdd
└─sdd1 ext4     43fbd170-3eff-40ec-95fc-c6cc090a5bc9
sde
└─sde1 ext4     fedd9ed1-d4cf-4d3c-b105-7b3296f157b4
```

In this case, `/dev/sda1` is the 1TB USB hard drive with `ext4` format, `/dev/sdc` is the 2TB SATA drive with `ext4` format, `/dev/sdd1` is the 4TB USB hard drive with `ext4` format, and lastly `/dev/sde1` is the 8TB USB drive. We'll need the UUIDs later, so copy them somewhere.

## Preparing the Folders

We'll mount the hard drives in `/media`, for this we'll create the required folders like so:

```bash
sudo mkdir /media/sata_2tb /media/usb_1tb /media/usb_4tb /media/usb_8tb
```

You may also change the permissions on these folders if it causes any problem.

```bash
sudo chmod -R 777 /media/sata_2tb /media/usb_1tb /media/usb_4tb /media/usb_8tb
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
UUID=e8f133c2-af07-4a7a-a42f-21f857a9a06f /media/usb_1tb ext4 defaults 0 0
UUID=418987da-2351-11e9-aec8-b8975ad7798b /media/sata_2tb ext4 defaults  0 0
UUID=fedd9ed1-d4cf-4d3c-b105-7b3296f157b4 /media/usb_4tb ext4 defaults 0 0
UUID=43fbd170-3eff-40ec-95fc-c6cc090a5bc9 /media/usb_8tb ext4 defaults 0 0
```

Finally, reboot the server. The hard drives should now be automatically mounted.
