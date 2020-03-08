## Enable the micro-usb connector as a host

The micro-usb connector is usb0. Per default it is in otg mode.
https://forum.armbian.com/topic/4814-orange-pi-one-usb-otg/
discusses what is needed.

You need to tweak DT by changing the "dr-mode" to "host" since it is defaulted to "otg" for usb-gadget.
Decompile the DTB using "dtc" compiler, edit the resulting source by changing "dir-mode" to "host", 
and recompile the DTB
```
cd /boot/dtb
dtb=sun8i-h3-orangepi-one.dtb           # for orangepi-one
dts=$(basename $dtb .dtb).dts
cp $dtb $dtb.old
dtc -I dtb -O dts -o $dts $dtb
grep dr_mode $dts
                        dr_mode = "otg";

sed -i -e 's/dr_mode = "otg";/dr_mode = "host";/g' $dts
dtc -I dts -O dtb -o $dtb $dts
# can we ignore ca 200: Warnings (.._property): ... cell .. is not a phandle reference
reboot
```
Plug in a USB power monitor. It should no light up early during reboot (ca. 8 sec).

```
[Sun Mar  8 21:07:49 2020] rtl8192cu: Board Type 0
[Sun Mar  8 21:07:49 2020] rtl_usb: rx_max_size 15360, rx_urb_num 8, in_ep 1
[Sun Mar  8 21:07:49 2020] rtl8192cu: Loading firmware rtlwifi/rtl8192cufw_TMSC.bin
[Sun Mar  8 21:07:49 2020] ieee80211 phy0: Selected rate control algorithm 'rtl_rc'
[Sun Mar  8 21:07:49 2020] usbcore: registered new interface driver rtl8192cu
[Sun Mar  8 21:07:49 2020] rtl8192cu 1-1:1.0 wlx0013ef6d01cf: renamed from wlan0
```

Yeah, and `iwconfig` now shows a new device

```
wlx0013ef6d01cf  IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr=2347 B   Fragment thr:off
          Encryption key:off
          Power Management:off
```

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

