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
| Display | 13.3 inch 16:9, 2160x1440 (2K) | 💚 Yes | resolution is correctly detected by `xrandr`, backlight control does not work via native function keys, but works via additional scripts (TODO: document) |
| Storage | LITEON CB1-SD256, 256 GB | 💚 Yes | via standard kernel driver (TODO: document) |
| Soundcard  | Intel Kaby Lake-U/Y PCH - High Definition Audio with Dolby ATMOS | 💚 Yes  | via standard kernel driver, it also works fine with `pulseaudio` (TODO: document) |
| Speakers  | "Dolby ATMOS" | 👁️‍🗨️ Kinda | Right now only left speaker works, but it is not noticable as the sound feels quite "centered". I'm hoping a kernel update or some other fix for it will come out eventually to address this. | 
| Ports | 1 USB 3.0 / 3.1 Gen1, 1 USB 3.1 Gen2 |  💚 Yes | [USB-PD](https://en.wikipedia.org/wiki/USB_PD) works only  via left port, but it is a hardware limitation of the laptop | 
| Fingerprint Reader | proprietary sensor made by Huawei | 🚫 No | It is located on the power button, which itself is fully functional  |
| Wifi | Intel Dual Band Wireless-AC 8265 (a/b/g/n/ac) | 💚 Yes | requires kernel 4.12 and firmware from Debian Testing (TODO: document) | 
| Bluetooth | ? | ⚠️ Not tested | (TODO: test and document) |
| Battery | 40 Wh Lithium-Polymer | 💚 Yes | Everything works: current status, chargin/discharging rate and remaining battery time estimates |
| Lid | ACPI-compliant |  💚 Yes | Works as expected: I can just close the lid and it sleeps  |
| Keyboard |  | 👁️‍🗨️ Mostly | some function keys do not work (eg. display brightness control, keyboard backlight control) | 
| Touchpad | | 💚 Yes | Tap-to-click can be enabled via `libinput` (TODO: document) |
| Port Extender | USC-C dongle included with laptop | 💚 Yes | Full-size HDMI works as expected |

## Power Management

Been testing it with Debian Stable + backported kernel 4.12. I've installed laptop-mode-tools (instead of TLP) and minimalist i3-wm (instead of Gnome).  With this setup and my workflow (mostly browser + ssh) the battery lasts for around 6-7 hours. Switching to more intensive things like code compilation or video encoding for prolonged time cuts battery to around 4 hours. 

TODO: document `laptop-mode-tools`
