# RPI ESXi
VMWare ESXi installation on RPI

## Update the RPI using a base image 

Create an Image

Check the EEPROM is at the latest version

```
sudo rpi-eeprom-update
```

Update the EEPROM to the latest version

```
sudo rpi-eeprom-update -a
```

## Update RPI Firmware 

#### Erase the SD-Card used to boot the RPI

#### Download
Latest Raspberry Pi Firmware: [Link](https://github.com/raspberrypi/firmware/archive/master.zip)
UEFI Raspberry Pi Firmware: [Link](https://github-production-release-asset-2e65be.s3.amazonaws.com/224524130/cdfd4100-ec65-11ea-891d-df0d29919d12?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20201028%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20201028T212953Z&X-Amz-Expires=300&X-Amz-Signature=ace9e277c110ff9656092b5456029eaa277af03dd00e09add9d994cbb53966c5&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=224524130&response-content-disposition=attachment%3B%20filename%3DRPi4_UEFI_Firmware_v1.20.zip&response-content-type=application%2Foctet-stream)

## Extract the Files

#### Firmware-Master
Go to the directory `firmware-master->boot`
Copy all the files in that directory to the SD-Card erased in the previous steps

#### SD-Card
On the SD-Card - remove the four kernel files
- [x] kernel.img
- [x] kernel7.img
- [x] kernel7l.img
- [x] kernel8.img


#### RPi4_UEFI
Copy all the files

#### SD-Card
Paste the copied files - allow overwrite of existing files
Edit config.txt and add the following at the end of the file

```
gpu_mem=16
```
Save and quit

## Download VMWare ESXi ARM ISO

ESXi [Link](https://flings.vmware.com/esxi-arm-edition)

Create a VMWare account
