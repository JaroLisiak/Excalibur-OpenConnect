FROM ubuntu:16.04

MAINTAINER jaroslav.lisiak@gmail.com

RUN apt update
RUN apt install -y libgnutls28-dev \
libev-dev \
libwrap0-dev \
libpam0g-dev \
liblz4-dev \
libseccomp-dev \
libreadline-dev \
libnl-route-3-dev \
libkrb5-dev \
make-guile \
libtalloc-dev \
wget \
nano \
libprotobuf-c-dev \
libhttp-parser-dev \
libhttp-parser2.1 \
make-guile \
libtalloc-dev \
htop

## Radcli installation ##

RUN	mkdir /usr/local/src/radcli
RUN	wget -P /usr/local/src/radcli/ https://github.com/radcli/radcli/releases/download/1.2.10/radcli-1.2.10.tar.gz
RUN	tar -C /usr/local/src/radcli/ -zxvf /usr/local/src/radcli/radcli-1.2.10.tar.gz
RUN	cd /usr/local/src/radcli/radcli-1.2.10/
RUN	/usr/local/src/radcli/radcli-1.2.10/configure --sysconfdir=/etc/
RUN make
RUN make install


# Setup radcli files
RUN	sed -i "s|#nas-identifier\tmy-server-name|nas-identifier excalibur-vpn-server|g" /etc/radcli/radiusclient.conf
RUN	sed -i "s|authserver \tlocalhost|authserver \t172.18.0.3|g" /etc/radcli/radiusclient.conf
RUN	sed -i "s|acctserver \tlocalhost|acctserver \t172.18.0.3|g" /etc/radcli/radiusclient.conf
RUN	sed -i "s|radius_timeout\t10|radius_timeout\t30|g" /etc/radcli/radiusclient.conf

# Radius client (Openconnect server) + shared-secred
RUN	echo "172.18.0.3\ttesting123" >> /etc/radcli/servers

# Setup pkgconfig file
RUN	cp /usr/local/lib/pkgconfig/radcli.pc /usr/share/pkgconfig/radcli.pc
RUN rm /usr/local/lib/pkgconfig/radcli.pc





## Openconnect server installation ##

RUN	mkdir /usr/local/src/ocserv
RUN	wget -P /usr/local/src/ocserv/ ftp://ftp.infradead.org/pub/ocserv/ocserv-0.12.1.tar.xz
RUN	tar -C /usr/local/src/ocserv/ -xvf //usr/local/src/ocserv/ocserv-0.12.1.tar.xz
RUN	cd /usr/local/src/ocserv/ocserv-0.12.1/
RUN	/usr/local/src/ocserv/ocserv-0.12.1/configure --sysconfdir=/etc/
RUN	make
RUN	make install
RUN 	mkdir /usr/local/etc/ocserv
RUN 	cp /usr/local/src/ocserv/ocserv-0.12.1/doc/sample.config /usr/local/etc/ocserv/ocserv.config

# Edit config file
RUN	sed -i "s|auth = \"plain\[passwd=.\/sample.passwd\]\"|auth = \"radius \[config=\/etc\/radcli\/radiusclient.conf,groupconfig=true]\"|g" /usr/local/etc/ocserv/ocserv.config

# Copy dictionary file to right folder
RUN cp /usr/local/src/radcli/radcli-1.2.10/etc/dictionary /etc/radcli/dictionary

RUN ldconfig

