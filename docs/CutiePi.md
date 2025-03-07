# Raspberry Pi

Here you can find all the projects I did or am currently working on with my little Raspberry Pi, aka "CutiePi"!
I am currently running most of them as Docker Containers, but this page is dedicated to the bare metal installs.

## Operating System & Troubleshooting
I use Raspberry Pi OS Lite, a Debian ARM based distro and the official OS from the Raspberry Pi Foundation.
I used to have DietPi OS installed, but ran into problems after the Debian Bookworm release. I couldn´t boot my Pi anymore, so I put the SD card into my laptop
and mounted it to a new directory. To do this, I fist created the directory with `mkdir /mnt/pi` and then ran `lsblk` to find out the device name of the SD card. Then I mounted it to the directory with `sudo mnt /dev/sdX /mnt/pi`. I got a 'wrong fs type' error, so I used `mkfs -t ext4 /dev/sdX` to create the ext4 filesystem on it. It worked an I could now mount the card to the /mnt/pi directory and access all my files.
Of course you can also just reinstall from backup, however I found this method interesting and like the flexibility that having your entire OS on an SD card offers.
I then installed Raspberry Pi OS Lite on the SD card and enabled SSH on it, to copy all my config files to the Pi from my Laptop, using the 'secure copy' command:`scp /mnt/pi/neededfiles lily@cutiepi:/home/lily/configfiles`.
To clean up on my laptop I used the `sudo umount /mnt/pi` and `sudo rm -rf /mnt/pi` commands.

## DNS
### PiHole
I use PiHole as a DNS-based blocker from ads and malicious sites/tracking sites.
Another useful feature is the oprion to configure local DNS A and CNAME records.
I installed PiHole following <a href="https://docs.pi-hole.net/main/basic-install/" target="_blank">the official guide</a>. I wanted to run the web interface on a custom port, so I ran `sudo nvim /etc/lighttpd/lighttpd.conf` and searched for `server.port`. You can this value to the desired port, then access it in the browser with `http://<hostname>:<port>/admin`.
I added new Adists from <a href="https://github.com/RPiList/specials/blob/54876178ffa7e4d1224ac81b00bedd0040f65802/Blocklisten.md" target="_blank">here</a>, then updated Gravity (`pihole -g`).
I added a script and cronjob to delete the FTL database every week (`crontab -e 0 0 * * 0 FTLdb.sh`).
```bash
#!/bin/bash

sudo systemctl stop pihole-FTL 
sudo mv /etc/pihole/pihole-FTL.db /home/lily/scripts/pihole-FTL_$(date +"%y-%m-%d").db #remove original DB file, create a backup file with time stamp in my scripts directory

cd /home/lily/scripts
find . -name "pihole-FTL_*.db" -type f -mtime +7 -exec rm {} \; #remove all files containing "pihole-FTL_" older than a week
```
### Unbound DNS
I use Unbound as a recursive DNS Server. 
I configured my Pi as local DNS Server on my router, so all new clients in my network automatically use it.
I followed the installation instructions from <a href="https://docs.pi-hole.net/guides/dns/unbound/" target="_blank">here</a> and added it as a custom DNS Server in the PiHole DNS configuration.
### DNSSEC
I enabled the following security options in `/etc/unbound/unbound.conf.d/`:
```
server:
    harden-below-nxdomain: yes
    harden-referral-path: yes
    harden-algo-downgrade: no
    use-caps-for-id: no
    hide-identity: yes
    hide-version: yes
```
To generate DNSSEC keys, run `sudo unbound-control-setup`, then restart unbound with `sudo systemctl restart unbound.service`.
Using `dig +dnssec A www.dnssec.cz` you can test if DNSSEC works. The result should contain the `ad` parameter.

## VPN Server
I used PiVPN to turn my Pi into a VPN Server with Wireguard protocol. Now I can establish a secure connection into my home network at any time.
After the inital installation, clients can be added with `pivpn -a`. I added my phone and laptop as a client.
I downloaded the Wireguard App on my phone, then generated a QR code with `pivpn -qr` to establish the VPN tunnel.
For my laptop, I copied the .conf file generated by PiVPN to my laptop using secure copy:
`scp lily@cutiepi:configs/laptop.conf ~/pivpn`.
I changed the Wireguard default port, as some networks block default VPN ports. It can be done either in the PiVPN installation wizard or later in the `/etc/wireguard/wg0.conf` `/etc/pivpn/setupVars.conf` files, but make sure to do this before generating the config files for the clients!
Also make sure to enable port forwarding for the Pi on your router, so the port is accessible from outside the local network.

### DynDNS
As my ISP changes my public IP address, I need a dynamic DNS client to make sure I can always reach my VPN Server.
I registered with 'NoIP' for a DynDNS Service and then configured it on my router, so my home network has a domain name it can always be reached on, even if the IP address changes.

## Syncthing
I use Syncthing to sync files from my Pi to my Laptop, which is especially handy via VPN.
I installed Syncthing on my Pi and my Laptop. I could then access Syncthing on my Laptop browser via `http://localhost:8384/`.
Then I configured a folder to sync with my Pi and tried to sync the devices with the 'Add device' button.
As my Pi is headless, I thought I could simply add it via Device ID (I looked at `man synthing`, and you can display it with `syncthing --device-id`).
But it didn´t work, the logs said the connection was refused by my Pi.
I then set a GUI for my Pi´s Syncthing with `syncthing --gui-address=<host-ip>:8384`, so I could then access `cutiepi:8384` from my laptop's browser, and configured the rest from here.
I needed to change the default user and password on both Syncthing instances in order to sync them.

## Some notes on SSH
I would strongly recommend taking a look at SSH, as you will be using it a lot in a headless setup.
I made sure to change modify the default SSH settings provided in `/etc/ssh/sshd_config`. I uncommented and changed `Port` to a custom port and `PasswordAuthentication` as well as `PermitRootLogin` to 'no'.
Of course, when disabling Password Authentication, you need to make sure you have SSH keys setup with all devices you will connect to the Pi with.
A good instruction for SSH key setup can be found <a href="https://www.linode.com/docs/guides/use-public-key-authentication-with-ssh/" target="_blank">here.</a>
I also made a SSH config file on my clients so it's less work connecting to multiple servers, as you don't need to remember all the custom ports and so on: `nvim ~/.ssh/config` and then create entries in the following format:
```
Host CutiePi
    HostName 192.168.10.xx
    User Lily
    IdentityFile ~/.ssh/lilyskey #enter the path to the private key
    Port xxx #enter the custom SSH Port you set

Host NAS
    HostName 192.168.10.55
    ...
```
Your client will now use these parameters to connect via SSH to these hosts.
## Managing disk space
- Useful commands to manage the Pi's disk space are `df -h`and `ncdu /`. The latter provides a nice overview of all files and their sizes in the specified directory (in this case, root.)
- To view a script I wrote to monitor the disk space and send notifications to my phone if it's over the threshold, see [here](automation.md#monitoring-disk-space)
- Also make sure to run the apt `autoclean` and `autoremove`commands if you want to remove old packages, and package dependencies that are no longer needed.
- I also wanted to limit the space my journal log was taking up. For this, you can run `sudo journalctl --vacuum-time=2weeks` for example, to remove records older than two weeks.
    - You can also specify the maximal usage, free space, files and file size allowed for your journal by editing `/etc/systemd/journald.conf`, eg. by uncommenting and setting the line `SystemMaxFileSize=40M`. Then restart the service using `sudo systemctl restart systemd-journald` for the changes to take effect.
### Log Rotation
You can use `logrotate` to automise log management, for example compressing logs, and deciding how long they should be kept and deleting older logs.
You can do this by creating a `logrotate.conf` file (for all logs, or seperate .conf files for each log), which you fill with the options provided by `man logrotate`.
For example:
```bash
/var/log/*.log {
# rotate log files weekly
weekly
# keep 4 weeks worth of backlogs
rotate 4
# create new (empty) log files after rotating old ones
create
# use date as a suffix of the rotated file
dateext
# compress log files
compress
#max size
size 50M
#rm rotated logs older than <count> days
maxage 30
# packages drop log rotation information into this directory
include /etc/logrotate.d
}
``` 
You can then run  `logrotate -v logrotate.conf` to execute the log rotation and check on the process, and of course create a cronjob for this.