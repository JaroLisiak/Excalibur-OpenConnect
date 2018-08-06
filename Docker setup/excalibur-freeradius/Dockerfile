FROM ubuntu:16.04

RUN apt update
RUN apt install -y libwww-perl \
 nano \ 
 libconfig-inifiles-perl \ 
 libdata-dump-perl \
 libtry-tiny-perl \
 freeradius \
 libjson-perl \
 liblwp-protocol-https-perl

#ADD . /jaro

RUN	sed -i "s|FREERADIUS_OPTIONS=\"\"|FREERADIUS_OPTIONS=\"-t\" |g" /etc/default/freeradius


RUN rm /etc/freeradius/users
RUN   echo "DEFAULT Auth-Type := Perl" >> /etc/freeradius/users

RUN	sed -i "s|module = \${confdir}/example.pl|module = /etc/freeradius/excalibur-radius.pm |g" /etc/freeradius/modules/perl

COPY files/rlm_perl.ini /etc/freeradius/
COPY files/excalibur /etc/freeradius/sites-enabled/
COPY files/excalibur-radius.pm /etc/freeradius/

RUN	sed -i "s|URL = https://localhost/validate/check|URL = https://staging.xclbr.com/api/authorize \n TIMEOUT = 30 |g" /etc/freeradius/rlm_perl.ini

RUN   echo "client x.x.x.x { \n\tipaddr = x.x.x.x \n\tshortname = vpn \n\tsecret = testing123 \n}\n" >> /etc/freeradius/clients.conf

RUN	rm /etc/freeradius/sites-enabled/default
RUN	rm /etc/freeradius/sites-enabled/inner-tunnel

###asdasda