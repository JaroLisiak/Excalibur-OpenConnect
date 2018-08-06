FROM ubuntu:16.04

MAINTAINER jaroslav.lisiak@gmail.com

RUN 	apt update
RUN 	apt install -y libwww-perl \
 nano \ 
 libconfig-inifiles-perl \ 
 libdata-dump-perl \
 libtry-tiny-perl \
 freeradius \
 libjson-perl \
 liblwp-protocol-https-perl


RUN	sed -i "s|FREERADIUS_OPTIONS=\"\"|FREERADIUS_OPTIONS=\"-t\" |g" /etc/default/freeradius

# Set default auth type to Perl
RUN 	rm /etc/freeradius/users
RUN   	echo "DEFAULT Auth-Type := Perl" >> /etc/freeradius/users

# Set path to perl module
RUN	sed -i "s|module = \${confdir}/example.pl|module = /etc/freeradius/excalibur-radius.pm |g" /etc/freeradius/modules/perl

# Copy all necessary files
COPY files/rlm_perl.ini /etc/freeradius/
COPY files/excalibur /etc/freeradius/sites-enabled/
COPY files/excalibur-radius.pm /etc/freeradius/

# Set URL to Excalibur auth API
RUN	sed -i "s|URL = https://localhost/validate/check|URL = https://staging.xclbr.com/api/authorize \n TIMEOUT = 30 |g" /etc/freeradius/rlm_perl.ini

# Create entry for radius client
RUN   	echo "client 172.18.0.2 { \n\tipaddr = 172.18.0.2 \n\tshortname = vpn \n\tsecret = testing123 \n}\n" >> /etc/freeradius/clients.conf

# Remove unnecessary files
RUN	rm /etc/freeradius/sites-enabled/default
RUN	rm /etc/freeradius/sites-enabled/inner-tunnel

# Restart freeradius service
RUN 	service freeradius stop
CMD 	freeradius -f

