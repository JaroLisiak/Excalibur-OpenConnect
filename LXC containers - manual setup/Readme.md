# LXC containers - manual setup

## Containers
Following commands creates 2 containers with Ubuntu 16.04. Container named "OC" for **OpenConnect server** and "FR" for **Freeradius server**.
```bash
lxc launch ubuntu:16.04 OC
lxc launch ubuntu:16.04 FR
```
Make sure your LXD containers are available over the network (you answered "yes" during lxd init)
To show created containers enter:
```bash
lxc list
```
Remember IP addresses of both containers. We will use it during the setup.
## Freeradius container
We need enter to the FR container:
```bash
lxc exec FR bash
```
First we need to update system and install dependencies and freeradius server:
```bash
apt update
apt install libwww-perl libconfig-inifiles-perl libdata-dump-perl libtry-tiny-perl freeradius libjson-perl liblwp-protocol-https-perl freeradius
```
Complete all following instructions:
1. Download **excalibur** file from this repository to ***/etc/freeradius/sites-enabled/*** directory:
```bash
cd /etc/freeradius/sites-enabled/
wget https://raw.githubusercontent.com/JaroLisiak/Excalibur-OpenConnect/master/Docker%20setup/excalibur-freeradius/files/excalibur
```
2. Download **excalibur-radius.pm** file from this repository to ***/etc/freeradius/*** directory:
```bash
cd /etc/freeradius/
wget https://raw.githubusercontent.com/JaroLisiak/Excalibur-OpenConnect/master/Docker%20setup/excalibur-freeradius/files/excalibur-radius.pm
```
3. Download **rlm_perl.ini** file from this repository to ***/etc/freeradius/*** directory:
```bash
cd /etc/freeradius/
wget https://raw.githubusercontent.com/JaroLisiak/Excalibur-OpenConnect/master/Docker%20setup/excalibur-freeradius/files/rlm_perl.ini
```
4. In file ***/etc/default/freeradius*** change **FREERADIUS_OPTIONS=""** to **FREERADIUS_OPTIONS="-t"**.
5. In file ***/etc/freeradius/users*** remove all lines and add the line with: **DEFAULT Auth-Type := Perl**.
6. In file ***/etc/freeradius/modules/perl*** change **module = ${confdir}/example.pl** to **module = /etc/freeradius/excalibur-radius.pm**.
7. Add following lines to ***/etc/freeradius/clients.conf*** file:
```
client <OC-container-IP> {
	ipaddr = <OC-container-IP>
	shortname = OC server shortname
	secret = <shared-secret>
}
```
8. Remove unnecessary files:
```bash
rm /etc/freeradius/sites-enabled/default
rm /etc/freeradius/sites-enabled/inner-tunnel
```
Freeradius server is now configured and ready to handle all requests. To start freeradius server enter:
```bash
freeradius
```
When freeradius server won't start you need to stop it and try to run with -Xxx switches.
```bash
service freeradius stop
freeradius -Xxx
```
## OpenConnect container
We need enter to the OC container.
```bash
lxc exec OC bash
```
First we need to update system and install dependencies.
```bash
apt update
apt-get install libgnutls28-dev libev-dev libwrap0-dev libpam0g-dev liblz4-dev libseccomp-dev libreadline-dev libnl-route-3-dev libkrb5-dev make-guile libtalloc-dev
```

### Radcli
Download and install Radcli
```bash
mkdir /usr/local/src/radcli
cd /usr/local/src/radcli/
wget https://github.com/radcli/radcli/releases/download/1.2.10/radcli-1.2.10.tar.gz
tar zxvf radcli-1.2.10.tar.gz
cd radcli-1.2.10
./configure --sysconfdir=/etc/ && make && make install
```
Edit following lines in config file ***/etc/radcli/radiusclient.conf***
```bash
nas-identifier <Server-name>
authserver <FR-container-IP>
acctserver <FR-container-IP>
```
Add following line into ***/etc/radcli/servers*** file
```bash
<FR-container-IP>				<shared-secret>
```
Locate **radcli.pc** file
```bash
find / -type f -name "*.pc" |& grep -iv permission
```
Copy **radcli.pc** file to ***/usr/share/pkgconfig/*** folder
```bash
cp /usr/local/lib/pkgconfig/radcli.pc /usr/share/pkgconfig/radcli.pc
```
Remove old **radcli.pc** file
```bash
rm /usr/local/lib/pkgconfig/radcli.pc
```
### OpenConnect
Download and install OpenConnect server
```bash
mkdir /usr/local/src/ocserv
cd /usr/local/src/ocserv
wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.12.1.tar.xz
tar xvf ocserv-0.12.1.tar.xz
cd ocserv-0.12.1
./configure --sysconfdir=/etc/ && make && make install
```
Create directory for config files
```bash
mkdir /usr/local/etc/ocserv
```
Copy config file into created directory
```bash
cp /usr/local/src/ocserv/ocserv-0.12.1/doc/sample.config /usr/local/etc/ocserv/ocserv.config
```
#### Configuration
Comment all "auth" lines in ***/usr/local/etc/ocserv/ocserv.config*** file and insert following line
```bash
auth = "radius [config=/etc/radcli/radiusclient.conf,groupconfig=true]"
```
Copy **dictionary** file to right location
```bash
cp /usr/local/src/radcli/radcli-1.2.10/etc/dictionary /etc/radcli/dictionary
```
Run following command to update dynamic linker:
```bash
ldconfig
```
#### Run
Enter following lines to run Ocserv in debug mode
```bash
cd /usr/local/src/ocserv/ocserv-0.12.1/src/
./ocserv -d 9999 -f -c /usr/local/etc/ocserv/ocserv.config
```
Ocserv is now running in debug mode. When you exit container Ocserv will stop.

To run Ocserv in background run this command:
```bash
lxc exec  --mode=non-interactive OC -- /bin/bash -c "cd /usr/local/src/ocserv/ocserv-0.12.1/src/ && ./ocserv -f -c /usr/local/etc/ocserv/ocserv.config"
```
Now you can exit container and Ocserv will run inside the container.

## Important!

To forward ocserv requests into OC container we need to use following command:
```bash
iptables -t nat -A PREROUTING -p tcp --dport 5004 -j DNAT --to-destination <OC-container-IP>:443
```
Make sure you use IP address of your OpenConnect container. 
OpenConnect server is now listening on 5004 port.

