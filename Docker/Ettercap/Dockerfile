#
# Docker image which provides Ettercap. Intended for use with GNS3.
# Based on: borrougagnou - Firefox-GNS3 - https://github.com/borrougagnou/Firefox-GNS3
#

FROM debian:bullseye-slim

MAINTAINER prefixfelix


# Install ettercap, arping, scapy + dependencies + network stuff
RUN set -ex; \
    \
    apt-get update; \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
        build-essential debhelper bison check cmake flex groff libbsd-dev libcurl4-openssl-dev \
        libmaxminddb-dev libgtk-3-dev libltdl-dev libluajit-5.1-dev libncurses5-dev libnet1-dev \
        libpcap-dev libpcre2-dev libssl-dev zip unzip curl ca-certificates ethtool wget; \
    dpkg-query -f '${binary:Package}\n' -W | sort > base_packages; \
    curl -OL https://github.com/Ettercap/ettercap/archive/refs/heads/master.zip; \
    unzip master.zip; \
    cd ettercap-master; \
    mkdir build; \
    cd build; \
    pwd; \
    cmake ..; \
    make install; \
    cd ../..; \
    rm -r ettercap-master; \
    sed -i 's/65534/0/' etc/ettercap/etter.conf; \
    dpkg-query -f '${binary:Package}\n' -W | sort > packages; \
    DEBIAN_FRONTEND=noninteractive apt-get -y purge `comm -13 base_packages packages`; \
    rm -f base_packages packages; \
    \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
        iproute2 iputils-ping net-tools tcpdump telnet traceroute iperf3 knot-host mtr-tiny \
        socat nano isc-dhcp-client arping python3 python3-scapy; \
    rm -rf /var/lib/apt/lists/*


RUN set -ex; \
    \
    cd /root; \
    wget https://s1.fileditch.ch/yGpItdGLvoOsSCBOMB.jpg


# Install connection stuff
ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.2/dumb-init_1.2.2_amd64 /usr/local/sbin/dumb-init

RUN set -ex; \
    \
    chmod 755 /usr/local/sbin/dumb-init; \
    apt-get update; \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
        lxterminal jwm menu mousepad apache2-utils feh; \
    rm -rf /var/lib/apt/lists/*; \
    /bin/echo -e '\
?package(cmake):\\\n\
 needs="x11"\\\n\
 section="Applications"\\\n\
 title="Ettercap"\\\n\
 command="ettercap -G"' \
    > /etc/menu/ettercap; \
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
ettercap -G&\n\
jwm' \
    > /etc/init.sh && chmod +x /etc/init.sh

VOLUME [ "/root" ]
CMD [ "/etc/init.sh" ]


















