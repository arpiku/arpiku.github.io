---
title: Making tp-Link T2UB Archer Nano Work on NixOS 
author: arpiku 
date: 2024-04-14 00:00:00 +0530
categories: [Wifi, Tech-Doc, Engineering, NixOS]
tags: [NixOS, Drivers, Tech-Doc]
pin: false
---

# Making tp-Link T2UB Archer Nano Work on NixOS

Generally speaking Linux is fantastic when it comes to hardware support, most things just work out of the box, however sometimes
an obscure or perhaps a piece of hardware designed only for Windows or MacOS can result in a missing driver.
I faced this challenge with my tp-link T2UB Archer Nano dongle.
The thing worked out of the box on Ubuntu but for Pop-OS and other OS it required to me to manually compile and install the driver, more about which 
can be learned [here](https://raspberrypi.stackexchange.com/questions/17187/why-do-i-get-this-error-trying-to-install-tp-link-tl-wn725n-wireless-adapter-d/17188#17188).

However, as I drive NixOS as my main workstation OS these days, life is considerably easier, just add the following line to your configuration 
in ` /etc/nixos/configuration.nix ` or maybe in your flaked module.

```nix
  boot.extraModulePackages = [ config.boot.kernelPackages.rtl8821cu ];
```

After this once you reboot you should find the dongle working, however one problem that I did face was it being stuck on 2.4GHz frequency rather than
using the available 5GHz band.
To fix this we can use `iwconfig`, if you don't have it installed just add the `wirelesstools` package in your NixOS configurations system packages.

Using the command would result in something like this 
```bash
user@workstation ~> iwconfig
lo        no wireless extensions.
                                                                                                                                                                                                                 
eno1      no wireless extensions.
                                                                                                                                                                                                                 
wlp8s1f3u1i2  IEEE 802.11  ESSID:"NET-5G"
          Mode:Managed  Frequency:5.745 GHz  Access Point: 34:24:F3:12:4E:C2
          Bit Rate=234 Mb/s   Tx-Power=30 dBm
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
          Link Quality=51/70  Signal level=-59 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:83   Missed beacon:0
```
As you can see I have already set my frequency to 5GHz, in order to get here just execute the following commands in your terminal.

```
sudo iwconfig <interface name> freq 5.18G
```

Here the `interface_name` refers to the rather abstract idea of “connection” rather than the actual device, but for our case it is one and the same,
just select your device, in your case it may something like `wlan0` or `wlan1`,for me I got this weird looking name of `wlp8..i2` so the command would look like

`sudo iwconfig wlp8s1f3u1i2 freq 5.18G` 

And that's it!
You are set, you can connect to your 5GHz band now and surf the internet to your hearts content!



