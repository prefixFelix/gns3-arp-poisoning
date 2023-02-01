#
# Docker image which provides Filezilla client. Intended for use with GNS3.
# Based on: borrougagnou - Firefox-GNS3 - https://github.com/borrougagnou/Firefox-GNS3
#

FROM debian:bullseye-slim

MAINTAINER prefixfelix

# Install network stuff
RUN set -ex; \
    \
    apt-get update; \
    dpkg-query -f '${binary:Package}\n' -W | sort > base_packages; \
    dpkg-query -f '${binary:Package}\n' -W | sort > packages; \
    DEBIAN_FRONTEND=noninteractive apt-get -y purge `comm -13 base_packages packages`; \
    rm -f base_packages packages; \
    \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
        iproute2 iputils-ping net-tools tcpdump telnet traceroute iperf3 knot-host openssh-client mtr-tiny \
        socat nano vim-tiny vim isc-dhcp-client ca-certificates wget; \
    rm -rf /var/lib/apt/lists/*


RUN set -ex; \
    \
    cd /root; \
    wget https://s1.fileditch.ch/VzNiwPxctrGSnIksWRg.jpg


# Install connection stuff
ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.2/dumb-init_1.2.2_amd64 /usr/local/sbin/dumb-init

RUN set -ex; \
    \
    chmod 755 /usr/local/sbin/dumb-init; \
    apt-get update; \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
        filezilla lxterminal jwm menu mousepad apache2-utils feh; \
    rm -rf /var/lib/apt/lists/*; \
    /bin/echo -e '\
?package(filezilla):\\\n\
 needs="x11"\\\n\
 section="Applications"\\\n\
 title="Filezilla"\\\n\
 command="filezilla"' \
    > /etc/menu/filezilla; \
    echo "postrun="\""sed -i '/^    </ d' debian-menu"\" >> /etc/menu-methods/jwm; \
    sed -i 's/\(Desktops width\)="[0-9]*"/\1="2"/' /etc/jwm/system.jwmrc; \
    update-menus; \
    mkdir -p /root/.config/lxterminal; \
    /bin/echo -e '\
[general]\n\
scrollback=1000\n\
fgcolor=#ffffff' \
    > /root/.config/lxterminal/lxterminal.conf; \
    /bin/echo -e '\
\x23!/bin/sh\n\
[ $$ -eq 1 ] && exec dumb-init "$0" "$@"\n\
\n\
cd\n\
export SHELL=/bin/bash\n\
dhclient\n\
filezilla&\n\
jwm' \
    > /etc/init.sh && chmod +x /etc/init.sh


CMD [ "/etc/init.sh" ]
VOLUME [ "/root" ]
