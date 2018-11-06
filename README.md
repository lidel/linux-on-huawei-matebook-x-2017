# GNU/Linux on Huawei MateBook X (2017)

> Brain dump: MateBook X running Debian Linux

![](https://web.archive.org/web/20170805112512if_/https://www.notebookcheck.net/fileadmin/Notebooks/Huawei/MateBook_X/4zu3_huawei_matebook_x_teaser.jpg)

## Background

First Huawei MateBook X was released in 2017.
It came with proprietary Microsoft Windows 10 and there was very little information available on its Linux support. 

I am running Debian on it. This repository documents what works and what does not.

## Linux Support Matrix

| Device | Model |  Works | Notes |
| --- | --- |  :---: | --- |
| Processor | Intel Core i5-7200U | 💚 Yes | 4 cores (2 real ones), power states etc work out of the box (TODO: document)  |
| Graphics | Intel HD Graphics 620 |  💚 Yes | via standard kernel driver (TODO: document) |
| Memory | 8192 MB |  💚 Yes |  |
| Display | 13.3 inch 16:9, 2160x1440 (2K) | 💚 Yes | resolution is correctly detected by `xrandr`, backlight control does not work via native function keys, but works via additional scripts (see [Display Backlight](#display-backlight)) |
| Storage | LITEON CB1-SD256, 256 GB | 💚 Yes | via standard kernel driver (TODO: document) |
| Soundcard  | Intel Kaby Lake-U/Y PCH - High Definition Audio with Dolby ATMOS | 💚 Yes  | via standard kernel driver, it also works fine with `pulseaudio` (TODO: document) |
| Speakers  | "Dolby ATMOS" | 👁️‍🗨️ Kinda | Right now only left speaker works, but it is not noticable as the sound feels quite "centered". I'm hoping a kernel update or some other fix for it will come out eventually to address this. | 
| Ports | 1 USB 3.0 / 3.1 Gen1, 1 USB 3.1 Gen2 |  💚 Yes | [USB-PD](https://en.wikipedia.org/wiki/USB_PD) works only  via left port, but it is a hardware limitation of the laptop | 
| Fingerprint Reader | proprietary sensor made by Huawei | 🚫 No | It is located on the power button, which itself is fully functional  |
| Wifi | Intel Dual Band Wireless-AC 8265 (a/b/g/n/ac) | 💚 Yes | requires kernel 4.12 and firmware from Debian Testing (TODO: document) | 
| Bluetooth | Intel (idVendor:0x8087, idProduct:0x0a2b) | 💚 Yes | ([see details below](#bluetooth)) |
| Airplane Mode | Wifi+Bluetooth | 💚 Yes | ([see details below](#airplane-mode)) |
| Battery | 40 Wh Lithium-Polymer | 💚 Yes | Everything works: current status, chargin/discharging rate and remaining battery time estimates |
| Lid | ACPI-compliant |  💚 Yes | Works as expected: I can just close the lid and it sleeps  |
| Keyboard |  | 👁️‍🗨️ Mostly | Some function keys do not work (eg. display brightness control), but there is ongoing dev in [aymanbagabas/Huawei-WMI/issues/2](https://github.com/aymanbagabas/Huawei-WMI/issues/2) | 
| Touchpad | | 💚 Yes | Tap-to-click can be enabled via `libinput` ([see details below](#touchpad)) |
| Port Extender | USC-C dongle included with laptop | 💚 Yes | Full-size HDMI works as expected |

## Power Management

Been testing it with Debian Stable + backported kernel 4.12. I've installed laptop-mode-tools (instead of TLP) and minimalist i3-wm (instead of Gnome).  With this setup and my workflow (mostly browser + ssh) the battery lasts for around 6-7 hours. Switching to more intensive things like code compilation or video encoding for prolonged time cuts battery to around 4 hours. 

TODO: document `laptop-mode-tools`

## Touchpad

I prefer natural (reversed) scrolling and tap-to-click.
Both behaviors are disabled by default, but can be easily enabled via libinput.

To test, change a property manually:

```
$ xinput list | grep Touchpad
    ↳ ELAN2201:00 04F3:3056 Touchpad          	id=10	[slave  pointer  (2)]

$ xinput list-props 10 | grep Tapping\ Enabled
    libinput Tapping Enabled (280):	0

$ xinput set-prop 10 280 1

$ xinput list-props 10 | grep Tapping\ Enabled
    libinput Tapping Enabled (280):	1
```

To make it permanent, create `/etc/X11/xorg.conf.d/40-libinput.conf` with:

```
Section "InputClass"
        Identifier      "touchpad catchall"
        MatchIsTouchpad "on"
        Driver           "libinput"
EndSection

Section "InputClass"
        Identifier "tap-by-default"
        MatchIsTouchpad "on"
        MatchDriver "libinput"
        Option "Tapping" "on"
        Option "NaturalScrolling" "true"
        Option "AccelSpeed" "1"
        #Option "TappingButtonMap" "lmr"       
EndSection
```

## Display Backlight

Backlight control  may not work out of the box in userland tools such as `xbacklight`. 

To activate it, create `/etc/X11/xorg.conf.d/30-intel.conf` with:

```
Section "Device"
    Identifier  "Card0"
    Driver      "intel"
    Option      "Backlight"  "intel_backlight"
EndSection
```

Reload X11. Now `xbacklight +10` should increase brightness by 10%. Hardware keys do not work yet, but one can to bind this to any keyboard combination (eg. `Mod4` (Windows key) + `F1` (brightness key))

If you want to get visual feedback on every change consider using [backlight-ctrl.sh](https://github.com/lidel/dotfiles/blob/master/bin/backlight-ctl.sh).


## Bluetooth

Tested with a mouse  (MX Anywhere 2S) and worked as expected. 

Following `apt install bluetooth` from [BluetoothUser](https://wiki.debian.org/BluetoothUser) guide should be enough.
I also installed `blueman` and run `blueman-applet &` for a handy tray icon in i3 status bar. 

### Bluetooth Troubleshooting

Bluetooth registers itself as an USB device and default power saving settings may be too aggressive for wireless mouse and keyboard. In my case mouse was unable to wake up from autosuspend so for now I just disable it on boot:

```bash
# Prevents the Bluetooth USB card from autosuspending, which (as of this edit) borks it
# Include this somewhere it gets called at boot, for optimal effect; e.g. /etc/rc.local
# Credit: http://ubuntuforums.org/showthread.php?t=2159645&page=6&p=12926730#post12926730
 
BTUSB_DEV="8087:0a2b"
BTUSB_BINDING="$(lsusb -d "$BTUSB_DEV" |
  cut -f 1 -d : |
  sed -e 's,Bus ,,' -e 's, Device ,/,' |
  xargs -I {} udevadm info -q path -n /dev/bus/usb/{} |
  xargs basename)"
 
 
echo "Disabling autosuspend for Bluetooth USB device: $BTUSB_BINDING..."
echo -1 > "/sys/bus/usb/devices/$BTUSB_BINDING/power/autosuspend_delay_ms"
```


If you are using a power management solution, you may want to see instructions below:

#### TLP

> I use `tlp` for power management. I had to set `USB_AUTOSUSPEND=0` in `/etc/default/tlp` to prevent random outages of USB and bluetooth (requiring a cold restart to correct). [#](https://github.com/lidel/linux-on-huawei-matebook-x-2017/issues/5#issue-339338858)

#### Laptop Model

If you use `laptop-mode` for power management, set `CONTROL_BLUETOOTH=0` in `/etc/laptop-mode/conf.d/bluetooth.conf`.


## Airplane Mode

Works as expected.

`rfkill list` shows current status of radio interfaces:

```
 rfkill list                                                                      ~
0: phy0: Wireless LAN
	Soft blocked: no
	Hard blocked: no
11: hci0: Bluetooth
	Soft blocked: no
	Hard blocked: no
```

Toggling "airplane mode": `rfkill block all` / `rfkill unblock all`
