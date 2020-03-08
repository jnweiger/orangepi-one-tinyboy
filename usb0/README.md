## Enable the micro-usb connector as a host

The micro-usb connector is usb0. Per default it is in peripheral mode.
Switching to host mode, was easy with the 3.3.113 kernel. But the realtek wifi driver is not working well with this old kernel.
We update to the buster-next kernel, which is 4.19.62

But then enabling the port in `host` mode is more difficult:
https://forum.armbian.com/topic/4814-orange-pi-one-usb-otg/
discusses what is needed.  https://patchwork.kernel.org/patch/9632731/

    wget https://git.kernel.org/pub/scm/linux/kernel/git/arm64/linux.git/plain/arch/arm/boot/dts/sun8i-h3-orangepi-one.dts

This should toogle gpio pin PL2 to power the port -- it apparently does not work for me.

### Try toggle the pin manually

According to https://linux-sunxi.org/GPIO the gpio number as used in /sys/class/gpio is computed as
(position of letter in alphabet - 1) * 32 + pin number

PL2 is at (b'L'[0]-b'A'[0])*32+2 = 11*32+2 = 354

Double check the computation:
    grep PL2 /sys/kernel/debug/pinctrl/*/pinmux-pins
     /sys/kernel/debug/pinctrl/1f02c00.pinctrl/pinmux-pins:pin 354 (PL2): (MUX UNCLAIMED) 1f02c00.pinctrl:354

But
    echo 354 > /sys/class/gpio/export
    bash: echo: write error: Device or resource busy




apt-get install gpiod
gpioinfo
...
 gpiochip1 - 32 lines:
	line   0:      unnamed       unused   input  active-high
	line   1:      unnamed       unused   input  active-high
	line   2:      unnamed  "usb0-vbus"  output  active-high [used]
	line   3:      unnamed        "sw4"   input   active-low [used]
...
# Hmmm.
