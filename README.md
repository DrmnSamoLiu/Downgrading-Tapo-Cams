### I've seen so much people saying they can't perform the "Micro SD card downgrade" with their Tapo cams, so I decide to create this repo to make a more technical guide on how the mechanism work.
-----

### TL;DR
The micro SD card you use is key point here. It needs to be **clean and with a proper partition table**, or the cam will treat it differently and not use it during the "check for update file" process.


------

### The following screenshots are taken from a Tapo C210 v2, running firmware v1.3.11 (The latest firmware as of 2024/6/25)


The boot script that handles update from micro SD card is `/etc/init.d/check_upgrade` <br>
Notice how it relies on the exsistance of `/dev/mmcblk0p1`, and that `/dev/mmcblk0p1` is mounted at `/tmp/sdcard`. <br> <br>

![圖片](https://github.com/DrmnSamoLiu/Downgrading-Tapo-Cams/assets/36998819/65e5418f-1d00-43d3-bac3-243aa6a50063)



<br><br>

This is what you will see under `/dev` when using a "clean" micro SD card with one FAT32 partition:<br><br>
![圖片](https://github.com/DrmnSamoLiu/Downgrading-Tapo-Cams/assets/36998819/328770da-0b67-4a37-8159-44297bf554c6)



<br><br>

And this is what you will see under `/dev` when using a micro SD card **That was formatted by the cam**, with one FAT32 partition.<br>
Observer how the kernel identifies the micro SD card `mmcblk0` but not listing the partition `mmcblk0p1`:<br>

![圖片](https://github.com/DrmnSamoLiu/Downgrading-Tapo-Cams/assets/36998819/6d9acf97-721c-45ab-b269-786d74790660)


In this case, the micro SD card will be mount at `/tmp/mnt/harddisk_1` on boot and be used as the storage media for video recordings.<br>
(Thus not triggering the micro SD card update script.)
<br>

This behavior will remain even after you perform quick formatting of the micro SD card on a PC.<br>

I don't know what kind of black magic the devs are using here, my guess is that they omitted the partition table and directly create a FAT32 system right at offset `0x0` of the micro SD card while initializing the card.<br>
Why a simple quick formatting on PC would not solve this problem is beyond my knowledge and I'll leave it for filesystem experts to answer.
<br><br>

# Solution

### Warning: The following instructions contains steps that will erase disks. CHOOSE CAREFULLY WHAT DISK YOU ARE OPERATING ON.

Here I'll use Windows OS as example, as I suppose this is what most people are running. <br>
OSX and Linux users should be able to find tools that can perform similar jobs. (For example, `gparted`.)

### The key point is to **create a partition table** on your micro SD card. <br>
Unfortunatelly, it seems this can not be done under GUI of Windows. <br>
 There might be some 3rd party software that are capable of doing this, but I'd prefer using native tools that are already in the OS, so I'll use `diskpart` as example:

1. Take your micro SD card from the camera, **backup** whatever is inside that you don't want to lose.
2. Insert the micro SD card into your Windows PC.
3. Press `Win + R` and run `diskpart`
4. Follow the steps in the following screenshot. <br>
****CHOOSE CAREFULLY WHAT DISK YOU ARE OPERATING ON, these operations will wipe the disk you select!**** <br> <br>
![圖片](https://github.com/DrmnSamoLiu/Downgrading-Tapo-Cams/assets/36998819/edb2a361-9d2f-4eb3-b5ec-d547e57fb2a6)

5. You should now have a clean `FAT32` disk with proper partition table.
6. Copy the firmware you wish to use for down/upgrade to the disk and rename it `factory_up_boot.bin`
7. Insert the micro SD card into your Tapo cam and powerup, wait for the process to occur.
