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

[Sun Mar  8 04:09:30 2020] usbcore: registered new interface driver usbfs
[Sun Mar  8 04:09:30 2020] usbcore: registered new interface driver hub
[Sun Mar  8 04:09:30 2020] usbcore: registered new device driver usb
[Sun Mar  8 04:09:31 2020] sun4i-usb-phy 1c19400.phy: Couldn't request ID GPIO
[Sun Mar  8 04:09:31 2020] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[Sun Mar  8 04:09:31 2020] ehci-platform 1c1a000.usb: EHCI Host Controller
[Sun Mar  8 04:09:31 2020] ehci-platform 1c1a000.usb: new USB bus registered, assigned bus number 1
[Sun Mar  8 04:09:31 2020] ehci-platform 1c1a000.usb: irq 26, io mem 0x01c1a000
[Sun Mar  8 04:09:31 2020] ehci-platform 1c1a000.usb: USB 2.0 started, EHCI 1.00
...
[Sun Mar  8 04:09:31 2020] sun4i-usb-phy 1c19400.phy: Linked as a consumer to regulator.4
...
[Sun Mar  8 04:09:31 2020] usb_phy_generic usb_phy_generic.2.auto: usb_phy_generic.2.auto supply vcc not found, using dummy regulator
[Sun Mar  8 04:09:31 2020] usb_phy_generic usb_phy_generic.2.auto: Linked as a consumer to regulator.0

...
[Sun Mar  8 04:09:31 2020] usb0-vbus: disabling


bin2fex /boot/script.bin boot.script.bin.fex
# study https://linux-sunxi.org/Fex_Guide#Port_Definitions
[usbc0]
usb_used = 1						# 1=enable
usb_port_type = 2					# 2=OTG, 1=host-only, 0=device-only
usb_detect_type = 0					# 0=no-checking, 1=VBus/id-check
usb_id_gpio = port:PG12<0><1><default><default>	
usb_det_vbus_gpio = port:PG12<0><1><default><default>	# <INPUT>
usb_drv_vbus_gpio = port:PL02<1><0><default><0>		# <OUTPUT><NO_PULLUP_DOWN><_default_mA><LOW>
usb_host_init_state = 1
usb_restrict_gpio =
usb_restric_flag = 0
usb_restric_voltage = 3550000
usb_restric_capacity = 5
usb_regulator_io = "nocare"
usb_regulator_vol = 0
usb_not_suspend = 0

fex2bin boot.script.bin.fex /boot/script.bin

Unpacking armbian-firmware (20.02.4) ...
Replacing files in old package firmware-realtek (20190114-2) ...

cat /sys/bus/platform/devices/usb0-vbus/regulator/regulator.4/power/control
 auto

