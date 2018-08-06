# Excalibur Openconnect - Freeradius setup
## Init
Create custom network interface **exc0** (172.18.0.0/24)
```bash
docker network create --driver=bridge --subnet=172.18.0.0/24 exc0
```

## Freeradius container
Move to **excalibur-freeradius** directory

```bash
cd excalibur-freeradius
```
Build image from Dockerfile
```bash
docker build -t rad .
```
Run container in background
```bash
docker run --network=exc0 --ip 172.18.0.3 --name radius-container -itd rad
```
Container is now running in background with default configuration. To attach the container to edit configuration enter
```bash
docker attach radius-container
```
To leave attached container without terminating it use <kbd>CTRL</kbd>+<kbd>P</kbd>+<kbd>Q</kbd>

## Openconnect + radcli container
Move to **excalibur-openconnect** directory

```bash
cd excalibur-openconnect
```
Build image from Dockerfile
```bash
docker build -t vpn .
```
Run container in background
```bash
docker run -p 5009:443 --network=exc0 --ip 172.18.0.2 --cap-add=NET_ADMIN --device=/dev/net/tun --name vpn-container -itd vpn /bin/bash -c "cd /usr/local/src/ocserv/ocserv-0.12.1/src/; ocserv -f -c /usr/local/etc/ocserv/ocserv.config"
```
Container is now running in background with default configuration. Openconnect container is litening on port 5009. IP address of openconnect server is 172.18.0.2. To reach oppenconnect from internet use your **public-IP:5009**
To attach the container to edit configuration enter
```bash
docker attach vpn-container
```
To leave attached container without terminating it use <kbd>CTRL</kbd>+<kbd>P</kbd>+<kbd>Q</kbd>

### Additional informations
To remove ALL containers enter
```bash
docker rm -f $(docker ps -a -q)
```
To remove ALL images enter
```bash
docker rmi -f $(docker images -q)
```
To stop containers enter
```bash
docker stop radius-container
docker stop vpn-container
```
