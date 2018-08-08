

# Excalibur Openconnect - Freeradius setup
## Init

__Settings used in this example:__

**Ocser container IP:** 172.18.0.2

**Freeradius container IP:** 172.18.0.3

**Shared secret:** testing123

**Timeout:** 90

**Excalibur API address:** https://staging.xclbr.com/api/pushka


---
After cloning this repository & before first build you can change default settings:

 - IP address of freeradius server:
	 - file excalibur-openconnect/Dockerfile - lines 39, 40, 44
 - IP address of OpenConnectServer (freeradius client):
	 - file excalibur-freeradius/Dockerfile - line 35
 - Timeout:
	 -  file excalibur-freeradius/files/rlm_perl.ini - line 7
	 -  /etc/ocserv/ocserv.conf - line **auth-timeout = 240** (default)
 - Excalibur API address:
	 - file excalibur-freeradius/files/rlm_perl.ini - line 2
 - Shared-secret:
	 - file excalibur-openconnect/Dockerfile - line 44
	 - file excalibur-freeradius/Dockerfile - line 35

If you want to use default settings used in this tutorial, you don't need to make any changes in cloned files.

---
Create custom network interface and name it **exc0** (172.18.0.0/24):
```bash
docker network create --driver=bridge --subnet=172.18.0.0/24 exc0
```
To check creating of network was successful use:
```bash
docker network ls
```

# Freeradius container
Move to **excalibur-freeradius** directory:

```bash
cd excalibur-freeradius
```
Build image from Dockerfile:
```bash
docker build -t rad .
```
Run container in background with default config:
```bash
docker run --network=exc0 --ip 172.18.0.3 --name radius-container -itd rad freeradius -f
```
You can check or change configuration inside the container:
```bash
docker run --network=exc0 --ip 172.18.0.3 --name radius-container -itd rad bash
docker attach radius-container
#you are inside the container
....
# do whatever you want inside the container
freeradius -f # run freeradius with new config
# leave container with Ctrl + P + Q
```
To leave attached container without terminating it, use <kbd>CTRL</kbd>+<kbd>P</kbd>+<kbd>Q</kbd>

# Openconnect + radcli container
Move to **excalibur-openconnect** directory:

```bash
cd excalibur-openconnect
```
Build image from Dockerfile:
```bash
docker build -t vpn .
```
Run container in background with default configuration:
```bash
docker run -p 5009:443 --network=exc0 --ip 172.18.0.2 --cap-add=NET_ADMIN --device=/dev/net/tun --name vpn-container -itd vpn /bin/bash -c "cd /usr/local/src/ocserv/ocserv-0.12.1/src/; ocserv -f -c /usr/local/etc/ocserv/ocserv.config"
```
Container is now running in background with default configuration. Openconnect container is litening on port 5009. IP address of openconnect server is 172.18.0.2. To reach oppenconnect from internet use your **public-IP:5009**
You can check or change configuration inside the container:
```bash
docker run -p 5009:443 --network=exc0 --ip 172.18.0.2 --cap-add=NET_ADMIN --device=/dev/net/tun --name vpn-container -itd vpn bash
docker attach vpn-container
# you are inside the container
....
# do whatever you want inside the container
cd /usr/local/src/ocserv/ocserv-0.12.1/src/
ocserv -f -c /usr/local/etc/ocserv/ocserv.config 
# run Ocserv with new config
# leave container with Ctrl + P + Q
```
To leave attached container without terminating it, use <kbd>CTRL</kbd>+<kbd>P</kbd>+<kbd>Q</kbd>
# Result
After successful installation it should looks like this:
![alt text](https://raw.githubusercontent.com/JaroLisiak/Excalibur-OpenConnect/master/LXC%20containers%20-%20manual%20setup/files/git_files/result.jpg "Result after successful setup")

----
### Additional informations
To remove ALL containers enter:
```bash
docker rm -f $(docker ps -a -q)
```
To remove ALL images enter:
```bash
docker rmi -f $(docker images -q)
```
To stop containers enter:
```bash
docker stop radius-container
docker stop vpn-container
```
