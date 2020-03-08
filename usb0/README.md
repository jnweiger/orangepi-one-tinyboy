## Enable the micro-usb connector as a host 

The micro-usb connector is usb0. Per default it is in peripheral mode.
Switching to host mode, was easy with th 3.3.xx kernel. But the realtek wifi driver is not working well with this old kernel.
We update to the buster-next kernel, which is 4.13.16

But then enabling the port in `host` mode is more difficult:
https://forum.armbian.com/topic/4814-orange-pi-one-usb-otg/
discusses what is needed.  https://patchwork.kernel.org/patch/9632731/

    wget https://git.kernel.org/pub/scm/linux/kernel/git/arm64/linux.git/plain/arch/arm/boot/dts/sun8i-h3-orangepi-one.dts

This should toogle gpio pin PL2 to power the port -- it apparently does not work for me. 
Try toggle the pin manually.

