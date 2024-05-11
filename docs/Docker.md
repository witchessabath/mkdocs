# Docker Containers

After a while of installing software locally on my Pi, I realised the benefits of using containers instead, so I started using Docker.
This way, I have each application installed in a sandboxed environment for more security, all dependencies come with the container so there's no "But it works on my machine!" effect, and management is very convenient with multiple services that allow more comfortable updates, logging, monitoring, configuration and networking than if the applications were all installed locally.

## Homarr
I use Homarr to have a dashboard overview of the services I need to access on my Pi.
It also has cool API integrations, for example to display PiHole data.
< screenshot>
I installed it using Docker compose, then configured it via the GUI webinterface.
As I wanted to use a custom icon for the tab bar and on the site itself, I downloaded `dino.png` on my laptop and moved it to `~/homarr/icons`, which I mapped to the icon directory in homarr using the line `- ./icons:/app/public/imgs/logo` under the `volumes:` section of my Docker Compose file.

## Traefik

### docker-compose.yml and traefik.yml
I use Traefik as areverse proxy and loadbalancer.
I also configured self-signed SSL certificates with it, using the ACME protocol.
I did it before by using **certbot**, but ACME certificates don't need to be manually renewed.
I did this by registering the witchessabath Domain, and getting a Cloudflare API Key for TLS encryption.
In my Traefik Docker Compose file I then added:
```
environment:
      - "CLOUDFLARE_EMAIL=xx@xx.com"
      - "CLOUDFLARE_API_KEY=xxxx"
```
Traefik is configured in `traefik.yml`, which I mapped in my Docker Compose file (`- ./conf/traefik.yml:/etc/traefik/traefik.yml:ro`).
You can view my Traefik configuration file here.

### Labels
To use my self-signed TLS certificates for my Docker containers, I give them the following labels:
```
- traefik.enable=true
```
Note that the containers must be in the same Docker network as the Traefik container.
For non-Docker containers or services I didn't attach labels to (like Portainer), I configured it like this:
```
portainer:
    image: portainer/portainer-ce:latest
    command: -H unix:///var/run/docker.sock
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
```

## Portainer
I use Portainer to have an overview and comfortable GUI management interface for all my Docker containers.
I find creating and managing Docker networks, assigning labels to containers, as well as reviewing container logs more comfortable through Portainer.
It also enables you to connect to a console and act as root user in the container, therefore replacing the need to connect to the Docker host and executing the `docker -it` command.
Portainer can be installed by creating a volume (`docker volume create portainer_data`) and then the container for it (`docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest`).

## Paperless
I use Paperless as a document managagement system.
I and configured it using a Docker Compose file, and added an Environment file for more fine-grained configuraions with the entry `env_file: docker-compose.env`.
The database I use with it is Redis. (This is one of the benefits of using Docker Compose - simply start the file with `services:` and define multiple services in the file.)

## Nextcloud
I run a Nextcloud instance in a container.
For this, I use the Nextcloud container that contains an Apache image to run the webserver.
I installed it with `docker run -d -p 2525:80 --name nextcloud --restart=unless-stopped -v /var/lib/docker/volumes/nextcloud:/var/lib/docker/volumes/nextcloud nextcloud`.
Anything I put in the Nextcloud volume on my Pi will appear on `cutiepi:2525` on my webbrowser, after executing the command: `docker exec -u www-data nextcloud php occ files:scan --all`. I created an **alias** for this command in my .bashrc.

## Uptime Kuma
I use Uptime Kuma, a simple yet powerful monitoring tool, to receive notifications about my services.
You can configure lots of notification options, I chose Mail and Telegram notifications.
I used the following Docker Compose file to install it:
```
version: '3.3'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma-data:/app/data
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 3025:3001 
    restart: always
```
and then created the `uptime-kuma-data` directory in the same parent directory.
I gave Uptime Kuma read only rights to the Docker Socket, so I can configure the Pi as a Docker Host in Uptime Kuma, and comfortably monitor all my containers
with it.

## Watchtower
I use Watchtower to keep my Docker containers up to date.
Watchtower searches local or online repositories for newer versions of the installed containers, and then updates them with the exact same settings that were configured before.
Watchtower is itself a container installed with 
```
docker run -d \
  --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower
```