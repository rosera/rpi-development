# RPI Create Image
Debian installation on RPI


A quick overview of how to build a image.

Bill of materials

- [x] Raspberry Pi x 1
- [x] MicroSDHC Card (Minimum 8GB)
- [x] USB Charging Cable (Same amount as RPi)
- [x] Machine to creating a RPi Disk Image

## Create the Disk Image

- [x] Download a [Raspberry PI image](https://www.raspberrypi.org/downloads/) e.g. Raspbian Debian Image 
- [x] Take the Memory Card and install an operating system.

1. Download either torrent or the zip file to your machine
__Note:__ the location where the file is being downloaded

2. Insert the memory card into your host machine and confirm it is recognised. 

3. Take note of how the memory card is labelled as we will use this information later 

4. Locate the downloaded file on your host.  
5. Extract the file to an image file
6. Right click on the downloaded image to show the menu
7. Open with Disk Image Writer option.

8. In the image writer, select the memory card inserted to be used to write the image. 

## Configure the Image

Once the image has been written to the memory card there will be two volumes available:

* boot
* rootfs 

__NOTE:__ On ChromeOs share the folders with Linux to enable write access then look in the folder `/mnt/chromeos/removable`

1. Select the BOOT from the file manager:

Click on the boot folder to mount the image. Then right click in the folder window and select the “open in terminal” option.

In the Terminal window for the boot folder add a SSH file to the root of the boot folder

```
touch ssh
```

This ensure the SSH service is started

Close the open terminal window

__Note__: You can also change the hostname ...

2. Select the ROOTFS from the file manager:

Click on the rootfs folder to mount the image. Then right click on the folder window and select the “open in terminal” option

In the Terminal window for the rootfs folder. Enter the config directory
```
cd etc
```

Edit the hostname file (use whatever editor you like)
```
sudo vi(m) hostname
```
Change the hostname from the default raspberrypi to whatever you want - save the file

You can now eject the rpi filesystem (i.e. boot & rootfs) mounts using the file manager 
