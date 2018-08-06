FROM ubuntu:16.04

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
libtalloc-dev


RUN	mkdir /usr/local/src/radcli

RUN	wget -P /usr/local/src/radcli/ https://github.com/radcli/radcli/releases/download/1.2.10/radcli-1.2.10.tar.gz
RUN	tar -C /usr/local/src/radcli/ -zxvf /usr/local/src/radcli/radcli-1.2.10.tar.gz
RUN	cd /usr/local/src/radcli/radcli-1.2.10/
RUN	/usr/local/src/radcli/radcli-1.2.10/configure --sysconfdir=/etc/ 
RUN make
RUN make install



RUN	sed -i "s|#nas-identifier\tmy-server-name|nas-identifier excalibur-vpn-server|g" /etc/radcli/radiusclient.conf
RUN	sed -i "s|authserver \tlocalhost|authserver \tx.x.x.x|g" /etc/radcli/radiusclient.conf
RUN	sed -i "s|acctserver \tlocalhost|acctserver \tx.x.x.x|g" /etc/radcli/radiusclient.conf
RUN	sed -i "s|radius_timeout\t10|radius_timeout\t30|g" /etc/radcli/radiusclient.conf


RUN   echo "x.x.x.x\ttesting123" >> /etc/radcli/servers

RUN cp /usr/local/lib/pkgconfig/radcli.pc /usr/share/pkgconfig/radcli.pc
RUN rm /usr/local/lib/pkgconfig/radcli.pc





## Instalacia openconnect servera ##

RUN	mkdir /usr/local/src/ocserv

RUN	wget -P /usr/local/src/ocserv/ ftp://ftp.infradead.org/pub/ocserv/ocserv-0.12.1.tar.xz
RUN	tar -C /usr/local/src/ocserv/ -xvf //usr/local/src/ocserv/ocserv-0.12.1.tar.xz
RUN	cd /usr/local/src/ocserv/ocserv-0.12.1/
RUN	/usr/local/src/ocserv/ocserv-0.12.1/configure --sysconfdir=/etc/ 
RUN make
RUN make install
RUN mkdir /usr/local/etc/ocserv
RUN cp /usr/local/src/ocserv/ocserv-0.12.1/doc/sample.config /usr/local/etc/ocserv/ocserv.config

RUN	sed -i "s|auth = \"plain\[passwd=.\/sample.passwd\]\"|auth = \"radius \[config=\/etc\/radcli\/radiusclient.conf,groupconfig=true]\"|g" /usr/local/etc/ocserv/ocserv.config

RUN cp /usr/local/src/radcli/radcli-1.2.10/etc/dictionary /etc/radcli/dictionary

#auth = "plain[passwd=./sample.passwd]"
#auth = "radius [config=/etc/radcli/radiusclient.conf,groupconfig=true]"





