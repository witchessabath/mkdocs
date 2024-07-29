# Automation

## Scripts
I am working on improving my shell scripting, and making my life easier through automation is a great motivator to do that!
Here are some scripts I found useful:

### Keep Docker Compose containers running
This script will ssh into remote Docker host, check if any containers have exited, and if so start them up again.
Note: This works quite easily as I name my Docker Compose directories exactly like the containers.
I run this once a day (`crontab -e 0 12 * * * /home/lily/scripts/dockermonitor.sh`), you could also have it running in the background using `nohup ./dockermonitor.sh &`.
```bash
#!/bin/bash

REMOTE_HOST="lily@cutiepi.local"
KEYFILE="/home/lily/.ssh/cutiepi"

ssh -i $KEYFILE -o StrictHostKeyChecking=no $REMOTE_HOST << EOF 
exited_containers=$(docker ps --filter status=exited --format "{{.Names}}") 

if [[ -n "\$exited_containers" ]], then
    for container_name in \$exited_containers; do
        echo "Container \$container_name has exited"
        cd \$container_name || { 
            echo "Directory \$container_name not found"
            continue
        }

        docker compose up -d

        running_container=$(docker ps --filter "name=\$container_name" --filter "status=running" --format "{{.Names}}")
        if [[ -n "\$running_container" ]]; then
            echo "Container \$container_name is running again"
         else
            echo "Failed to start container \$container_name"
        fi

         cd
    done
else
    echo "No exited containers"
fi
EOF
```
### Monitoring disk space
This script will monitor the disk space on my servers, and send a notification to my phone if a certain threshold is reached, using the <a href="https://github.com/akusei/pushover-bash" target="_blank">pushover app.</a> I created the cronjob: `0 20 * * 2,5 /home/lily/scripts/diskmonitor.sh`, so it will run twice a week.
```bash
#!/bin/bash 

#define a threshold in megabytes                              
THRESHOLD=600                                                                                                                                                                              #define a directory to be monitored (in this case root)                                                   
target=/                                                                                                                                                                                   
#check disk usage in machine readable format, excluding proc directory to not include running processes, storing only the size number in the variable
usage=$(sudo du -sm --exclude="/proc" "$target" | awk '{print$1}')
#calculate remaining space until threshold is reached 
remaining=$((THRESHOLD - usage))
if [ "$usage" -gt "$THRESHOLD" ]; then
    echo "Warning, Threshold exceeded! Disk is at "$usage"M"
    #send pushover message to my phone (Pushover must be pre-configured)
    pushover -T "Warning: Disk Space" "Diskspace on $HOSTNAME is running out! Disk is at "$usage"M"
else
echo "There are "$remaining"M remaining"  
fi
```

### Check if processes are running
Simple script to quickly check for running processes.
```bash
echo "Enter process name: "   
read process

if pgrep "$process" > /dev/null 2>&1; then
    echo "Process is running"
else
    echo "Process is not running."
fi