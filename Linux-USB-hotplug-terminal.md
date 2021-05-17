# Hotplug USB serial console

USB hotplug configuration to get a login terminal on a keyboard and/or monitorless Linux computer using two USB-to-serial FTDI adapters with their TX and RX pins connected appropriately.

This setup is a bit different from the config needed if the USB serial adapter is to be permanently plugged in - how to set that up is described [here](https://www.rogerirwin.co.nz/open-source/enabling-a-serial-port-console/) and in other places.

This configuration is tested on recent versions of Ubuntu and on RaspberryPi OS. Other systemd based Linux distributions may also work if configured the same way. The USB-to-serial adapters tested are using the common FTDI FT232 chipsets and the `ftdi_sio` Linux driver but any that show up in the OS as `/dev/ttyUSB*` devices could possibly work without changes. 

The `ttyUSB` devices will be automatically created by the OS when a FTDI adapter is plugged in. To also enable the creation of login terminals we need to add this extra configuration.

## Hardware connections
```
USB -> FTDI1       FTDI2 <- USB
        TX    <->   RX
        RX    <->   TX
        GND   <->   GND
```
It does not matter if the FTDI adapters are 3.3V or 5V logic level type as long as they are the same. Also, don't connect VCC between them! Both ends are assumed to be using 115200 kbit settings.

## The terminal side (your client) 
On the terminal side (if you run Linux) you may need to make sure your user ID is in a certain group to get access to the USB tty device without using the root account. On Ubuntu this is the `dialout` group.

Use your serial terminal software of choice like `minicom` or use screen: `screen /dev/ttyUSB0 115200`

If you use Windows PuTTY can do serial communications.

On a Mac the created USB device will be called something like `/dev/tty.usbserial-A50285BI`. Using `screen` with this should work fine, usage = as with Linux above.

## Target computer setup

### systemd unit modification

Edit the file `/lib/systemd/system/serial-getty@.service` and change the `ExecStart` line to this:
```
ExecStart=-/sbin/agetty 115200 %I $TERM
```
You can even enable automatic login on the terminal when the USB adapter is plugged in:
```
ExecStart=-/sbin/agetty --autologin youruser 115200 %I $TERM
```
You may need to run `systemctl daemon-reload` to make systemd reload the config.

### udev rule addition + script

A hotplug rule needs to be added to the `UDEV` subsystem together with a script that will (re)start the login TTY when the FTDI adapter is plugged in.

Create the file `/etc/udev/rules.d/99-hotplug-usb-tty.rules` with the following content:
```
KERNEL=="ttyUSB*", RUN+="/usr/local/bin/reload-usb-tty.sh"
```

Create the corresponding script `/usr/local/bin/reload-usb-tty.sh`:
```bash
#!/bin/bash
DEVICE=${DEVPATH##*/}
# logger "$( basename $0 ): udev: ${ACTION} ${DEVPATH}"  # optional syslog message
if [ "${ACTION}" = "bind" ]; then
	systemctl start serial-getty@${DEVICE}.service
fi
```
Also make it executable: `chmod +x /usr/local/bin/reload-usb-tty.sh`

After this step a login terminal should be launched automatically on the FTDI adapter TTY device when it is plugged in and you should see a login prompt on the terminal:
```
Ubuntu 20.04.2 LTS d270 ttyUSB0

d270 login:

```
If you plug the cable into the remote computer before starting the terminal on your client you may need to press a key or two to get the other computer to refresh the login prompt.

### Automatic idle logout

To automatically logout an idle login terminal make sure to set the `$TMOUT` environment variable during login. This will only work on idle shell prompts, not if the terminal is for example running `top`. 

Example configuration: Create the file `/etc/profile.d/ttyusb-autologout.sh` with the following content:
```bash
# autologout ttyUSB logins, TMOUT in seconds
if [[ $(tty) == /dev/ttyUSB* ]]; then export TMOUT=600; fi
```

## Notes
* Enabling this, in particular with automatic login, may have security implications if used outside of your home lab. 
* Booting the target computer with the FTDI adapter plugged in will make systemd timeout while trying to activate the device. This will add a minute or two to the boot time. It might be possible to fix (not investigated). 
* How to quit `screen`? Press `ctrl+a`, then `k`. 

