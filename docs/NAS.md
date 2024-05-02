# Network Attached Storage

I used Openmediavualt (OMV) to make a NAS out of my Raspberry Pi 3b+, to which I simply attached a HDD.

## Preparations
- Install Raspberry Pi OS
- Install OMV with the installation script: https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install 
- Wipe the drive(s) you wish to connect to your Pi. I did it like this:
    - Connect drive to PC, find out the device name by using `sudo fdisk -l`
    - Use dd command to write zeros to the drive: `sudo dd if=/dev/null of=/dev/sbd2 bs=5M status=progress`

## Setup a SMB share