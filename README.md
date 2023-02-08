# ARP-poisoning Lab


![](misc/overview.png)

## Overview

This lab is intended to be a simplified representation of an enterprise network. This is divided into two networks each:  
Network 1 should resemble an office network used by the employees. Because clients fluctuate frequently, IP addresses are assigned by a DHCP server. Our victim and the attacker are also located in this network. The victim tries to download and upload data from an FTP server located on the second network. Only the company's servers are located in this second network, which is why all clients have been assigned a fixed IP address. The attacker now tries to establish a position between the victim and the FTP server so that the attacker can control all traffic between the two. To achieve this, he has three different ARP-poisoning tools at his disposal: Ettercap, Scapy and ARPing.

| Role     | Purpose    | Software                | Virtualization-typ                                       | MAC                                                          | IP                                                           |
| -------- | ---------- | ----------------------- | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Victim   | FTP-Client | FileZilla               | [Docker](https://hub.docker.com/r/prefixfelix/filezilla) | 00:11:11:00:11:11                                            | DHCP                                                         |
| Attacker | MITM       | Ettercap, Scapy, ARPing | [Docker](https://hub.docker.com/r/prefixfelix/ettercap)  | 00:11:11:00:22:22                                            | DHCP                                                         |
| Server   | FTP-Server | vsFTPd                  | [Docker](https://hub.docker.com/r/fauria/vsftpd/)        | 00:22:22:00:11:11                                            | 192.168.2.210                                                |
| Switch   | Switch     | Open vSwitch            | [Docker](https://hub.docker.com/r/gns3/openvswitch)      | -                                                            | -                                                            |
| Router   | Router     | OpenWrt                 | Qemu                                                     | eth1: 00:11:11:00:00:00, <br />eth2: 00:22:22:00:00:00, <br />*eth3: 00:33:33:00:00:00* | eth1: 192.168.1.1<br />eth2: 192.168.2.1<br />*eth3: 192.168.3.1* |

## Procedure

To show the process and effect of arp poisoning, two different attack modes can be used. One is simple sniffing and the other is traffic manipulation:

1. Start all nodes. It makes sense to start the router first, because it needs a few seconds before it is fully operational.
2. The attacker now starts Ettercap to begin the ARP-poisoning. First, he scans the network using the *Scan for Hosts* function. Then the victim and the router are selected as the target in the *Host list*. Now the attack only has to be started via the *MITM menu*.

### Sniffing

In this case, the victim uploads an image to the FTP server. The attacker captures this image, as well as the login data.

1. The attacker loads the filter *[extract-img.ef](ettercap-filters/extract-img)*, which is located in his root folder, via the menu *Filters > Load a filter*. Afterwards he only has to open the *Connections* tab and thus is ready for sniffing.
2. The victim now logs into the FTP server with the username **bob** and the password **trustno1**. He then uploads the image *innocent-cat.jpg*, which is located in his root folder.
3. The attacker should now see the login details in the Ettercap console output, as well as the message *Found image!*. The captured image is stored in the root folder and can be opened using *feh*. The entire traffic can either be inspected directly in ettercap or exported via the *Logging* menu.

### Manipulation

In this case, the victim uploads a text file. Before it is forwarded to the FTP server, the attacker first modifies it. This happens without any delay, on-the-fly, so to speak.

1. The attacker loads the filter *[manipulate-txt.ef](ettercap-filters/manipulate-txt)*, which is located in his root folder, via the menu *Filters > Load a filter*. Afterwards he only has to open the *Connections* tab and thus is ready for the manipulation.
2. The victim now logs into the FTP server with the username **bob** and the password **trustno1**. He then uploads the text file *payroll.txt*, which is located in his root folder.
3. The attacker should now see the login details again in the Ettercap console output, as well as the message *Manipulated packet!*. To verify that the tampered file is really stored on the server, the file of the victim and the server can be extracted from each docker volume and then compared using *diff*. The volume can be displayed by clicking on the respective machine in GNS3 and selecting *Show in file manager*. Another possibility would be that the victim downloads the file again and compares it on his machine with the original file.

## Installation

> :information_source: Tested with GNS3 2.2.37 under Linux native, as well as with the GNS3 VM using VirtualBox 7.0.6.

1. Install GNS3 by following these instructions: [Linux](https://docs.gns3.com/docs/getting-started/installation/linux) / [Windows](https://docs.gns3.com/docs/getting-started/installation/windows)

2. Download the file *arp-poisoning.gns3project* from the repo. 

3. Open GNS3 and select the previously downloaded file under *File > Import portable project*. Then select a desired location and press OK. The project will now be imported, thereby various Docker containers will be downloaded from Dockerhub, which is why the import can take a few minutes.

   > :information_source: If you are using the GNS3 VM, you must first make sure that it is started before importing the project. This can be seen on the right side under the *Server Summary*.

   > :information_source: The images for the containers of the victim and the attacker can also be built via the [Dockerfiles](docker/) in the repo itself. How to use them with GNS3 can be read [here](https://docs.gns3.com/docs/emulators/create-a-docker-container-for-gns3/).
