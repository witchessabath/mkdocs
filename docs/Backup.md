# Backup
## Borg Backup

### Local Repository
I use Borg Backup to backup my Laptop, PC and Pi.
I backup my Laptop and PC to a remotely hosted Nextcloud instance and an external hard drive, and my Pi to a USB drive.
I execute the backups via script:
```
#!/bin/sh
REPO="xxx"
SOURCE="xxx"
export BORG_PASSPHRASE="xxx"

borg create --verbose $REPO::$(date +%Y-%m-%d) $SOURCE -e /home/lily/Nextcloud/
borg prune --verbose --list $REPO --keep-daily=7 --keep-weekly=4 --keep-monthly=6
```

### Remote Repository

On my OMV NAS, I use Borg Backup to a remote repository.
There's a Borg Backup Plugin you can use to condigure it in the Web GUI, but I simply couldn't get it to work.
You can check if the Borg connection works by executing the following command on the Borg Server:
`borg info 'ssh://server@user:<sshport>/./test`.
This always worked, but OMV always gave me an error message about the connection not being able to be established.
The plugin basically just executes the commands to initialize a remote Borg Repo, so I simply did this on the command line.

#### Prerequisites
- Borg installed on both backup server and client
- SSH connection with SSH keys established from client to server
!!! Note
    Borg does not work with a SSH password, SSH keys must be used. Borg needs to know what keys to use, so set an environment variable for it: `export BORG_SSH='ssh -i /path/to/SSH_KEY`
    I would strongly command using a custom SSH command by changing SSH port for the server in the sshd_config file.
    A great instruction for creating SSH keys can be found <a href="https://www.linode.com/docs/guides/use-public-key-authentication-with-ssh/" target="_blank">here.</a>
#### Configuring the Repo
- Use the `borg info` command mentioned above to make sure the client can reach the backup server.
- Initialize the repo: `borg init --encryption repokey user@host:/path/to/repo` 
!!! Note
    Make sure to copy not only the passphrase but also the repokey!
- Create a backup: `borg create user@host:/path/to/repo`
- You can then use the same script backups as above and then create a cronjob for it.

## Timeshift Snapshots

I use Timeshift for system snapshots.
Simply install the 'timeshift' package and use `man timeshift` to see all available options.
I use rsync mode since my filesystem is ext4 - there's a btrfs mode if that's the filesystem you use - but only on systems having an Ubuntu-type subvolume layout (with @ and @home subvolumes).