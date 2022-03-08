# HOWTO Setup Wiregiard Server On VPS

Main resource is https://github.com/linuxserver/docker-wireguard

## Add user

Add new user

```
# adduser vpsadmin
# usermod -aG sudo vpsadmin
```

## Install docker

Install docker engine that is needed to run `wireguard` server using one of convinient ways, e.g:

```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

Other ways can be found here:
* https://docs.docker.com/engine/install/ubuntu/

Add `vpsadmin` user to `docker` group to allow it control `wireguard` server.

```
$ sudo usermod -aG docker vpsadmin
```

Relogin and check that docker is installed properly:

```
$ sudo docker run hello-world
```

## Install docker-compose
`docker-compose` is needed to run dockerized wireguard server.

```
$ sudo apt install docker-compose
```

## Install qrencode

`qrencode` utility is required to encode peer's configuration into QR-code.
```
$ sudo apt install qrencode
```

## Create docker-compose configuration

Wireguard configuration is stored into `/extra/wireguard/docker-compose.yml`.

```
$ sudo mkdir -p /extra/wireguard
$ chmod a+rw /extra/wireguard
```

Creat `/extra/wireguard/docker-compose.yml` with the following content:

```
---
version: "2.1"
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - SERVERURL=5.101.179.100 #optional
      - SERVERPORT=51820 #optional
      - PEERS=mylaptop,myphone #optional
      - PEERDNS=auto #optional
      - INTERNAL_SUBNET=10.9.0.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
    volumes:
      - /extra/wireguard/config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

Set `PEERS` variable to a comma-separated list of peers that will use this server (e.g. `PEERS=phone,laptop,desktop`).

Check that `volumes` contains mapping for peer configs.


## Add aliases

This command aliases help to control the server

```
echo "alias wg-peer-config='qrencode -t ansiutf8 -r'" >> ~/.bash_aliases
echo "alias wgdown='COMPOSE_FILE=/extra/wireguard/docker-compose.yml docker-compose stop'" >> ~/.bash_aliases
echo "alias wglogs='COMPOSE_FILE=/extra/wireguard/docker-compose.yml docker-compose logs -tf --tail="50" ' >> ~/.bash_aliases
echo "alias wgpull='COMPOSE_FILE=/extra/wireguard/docker-compose.yml docker-compose pull' >> ~/.bash_aliases
echo "alias wgup='COMPOSE_FILE=/extra/wireguard/docker-compose.yml docker-compose up -d' >> ~/.bash_aliases
```


## Run server

Pull docker image and run it. Configuration files for peers well be generated at image startup and will be placed to `/extra/wireguard/config`.

```
$ wgpull
$ wgup
```

## Show config for specific peer

```
wg-peer-config /extra/wireguard/config/peer_myphone/peer_myphone.conf
```
