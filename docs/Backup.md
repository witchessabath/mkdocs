# Borg Backup

I use Borg Backup to backup my Laptop, PC and Pi.
I backup my Laptop and PC to a remotely hosted Nextcloud instance, and my Pi to a USB drive.
I execute the backups via script:
```
#!/bin/sh
REPO="xxx"
SOURCE="xxx"
export BORG_PASSPHRASE="xxx"

borg create --verbose $REPO::$(date +%Y-%m-%d) $SOURCE -e /home/lily/Nextcloud/
borg prune --verbose --list $REPO --keep-daily=7 --keep-weekly=4 --keep-monthly=6
```
# Timeshift Snapshots
I use Timeshift for system snapshots.
