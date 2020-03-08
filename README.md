## Installation

### hardware setup
* Prepare a LAN network 10.42.0.1/24 
* Workstation and OrangePI are connected to that network.

### boot image
* Download a debian buster image from https://www.armbian.com/orange-pi-one/
* Insert a new micro sd card. The empty filesystem gets mounted.
* `sudo umount /dev/sdb1`
* `sudo dd if=Armbian_20.02.1_Orangepione_buster_current_5.4.20.img of=/dev/sdb bs=1M`
* Remove and reinsert the micro sd card. The file system gets mounted.
* If the network does not provide DHCP, assign a static address:
    * `sudo sed -i -e 's@exit 0@ifconfig eth0 10.42.0.22/24 up\nexit 0@' /tmp/$USER/*/etc/rc.local`
    * `sync; sudo umount /dev/sdb1`
* Remove the micro sd card and boot the orangepi one with it.
* Connect ethernet cable and patiently wait ca. 2h for `ping 10.42.0.22` to respond.

### initial login
* `ssh root@10.42.0.22` -- intitial root password is `1234`
* Check default route and namserver, if needed
    * `route add default gw 10.42.0.100`
    * `echo nameserver 8.8.8.8 >> /etc/resolv.conf`
* `apt-get update; apt-get upgrade; sync; reboot`

