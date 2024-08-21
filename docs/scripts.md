# Automation

I am working on improving my shell scripting, and making my life easier through automated monitoring is a great motivator to do that!

## Notifications
### Uptime Kuma
Currently I use [Uptime Kuma](Docker.md#uptime-kuma) as a monitoring tool. 
Some things I cannot monitor with Uptime Kuma, so I wrote scripts and included a push notification to Uptime Kuma in them, so I know when a certain part of a script failed.<br />
To do this, go to `Uptime Kuma > Add New Monitor > Monitor Type "Push"`. Scroll down and check the Box next to `Upside Down Mode`.
The new monitor displays the push URL at the top, simply copy it and add it to your script with `curl`. Now, when the script calls the Push URL, you will now it failed. 

!!! note
    Make sure the client knows the hostname of the service Uptime Kuma is running on as it's included in the URL, or simply replace with localhost if the script is running on the same server as Kuma.

### Healthchecks.io
You can also use <a href="https://healthchecks.io/" target="_blank">Healthcheck.io</a> to check if the scripts themselves ran by appending `curl --retry 3 https://hc-ping.com/your-uuid-here/$?` to them, or to monitor cronjobs.
Healthchecks can send you success or failure messages through your chosen integration, and it can even measure job execution time.

## Scripts

### Monitoring disk space
This script will monitor the disk space on my servers, and send a notification to my phone if a certain threshold is reached, using Uptime Kuma. <br />
I created a monitored cronjob so it will run twice a week (`crontab -e 0 20 * * 2,5 /home/lily/scripts/diskmonitor.sh && curl -fsS -m 10 --retry 5 -o /dev/null https://hc-ping.com/my-uuid`)
```bash
#!/bin/bash 

#define a threshold in megabytes                              
THRESHOLD=600                                                                                                                                                                              #define a directory to be monitored (in this case root)                                                   
target=/                                                                                                                                                                                   
#check disk usage in machine readable format, excluding /proc to not include running processes, storing only the size in the variable
usage=$(sudo du -sm --exclude="/proc" "$target" | awk '{print$1}')
#calculate remaining space until threshold is reached 
remaining=$((THRESHOLD - usage))
if [ "$usage" -gt "$THRESHOLD" ]; then
    echo "Warning, Threshold exceeded! Disk is at "$usage"M"
    #send push message to Uptima Kuma
    curl http://localhost:3025/api/push/EfH8ApJpQu?status=up&msg=OK&ping=
else
echo "There are "$remaining"M remaining"  
fi
```

### Keep Docker Compose containers running
This script will ssh into remote Docker host, check if any containers have exited, and if so start them up again.
Note: This works quite easily as I name my Docker Compose directories exactly like the containers.
I run this once a day (`crontab -e 0 12 * * * /home/lily/scripts/dockermonitor.sh`), you could also have it running in the background using `nohup ./dockermonitor.sh &`.
```bash
#!/bin/bash

REMOTE_HOST="lily@cutiepi.local"
KEYFILE="/home/lily/.ssh/cutiepi"

#SSH into the server and store exited containers in variable
ssh -i $KEYFILE -o StrictHostKeyChecking=no $REMOTE_HOST << EOF 
exited_containers=$(docker ps --filter status=exited --format "{{.Names}}") 

#if variable not empty > cd into the container directories and run the docker compose up command
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
            #if containers can't be restarted, send push notification to Uptime Kuma
            echo "Failed to start container \$container_name"
            curl http://localhost:3025/api/push/bCe0PPb4OZ?status=up&msg=OK&ping=
        fi

         cd
    done
else
    echo "No exited containers"
fi
EOF
```

### Monitoring processes
Simple script to quickly check for running processes.
```bash
#!/bin/bash
echo "Enter process name: "   
read process

if pgrep "$process" > /dev/null 2>&1; then
    echo "Process is running"
else
    echo "Process is not running."
fi
```