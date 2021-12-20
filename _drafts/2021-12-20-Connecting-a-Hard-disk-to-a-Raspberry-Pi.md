---
published: false
---
A Raspberry Pi can be a quick solution to do some prototyping but if you need additional storage, an external hard drive can be very handy.

I was looking for a tool to store security camera feeds using a Raspberry Pi.
Even if you're just storing footage for a couple of days, an SD-Card is not a good option.
It's not so fast and it's could fail overtime due to lots of write operations.

Following steps describe how to connect an external USB Hard Drive/SSD to a Raspberry Pi.
I'm using an old Raspberry Pi 3 which unfortunately does not support booting from an USB Drive.
So I still need to boot the Pi using a small SD Card (2Gb works just fine).

Here are the steps I took:

- First prepare an SD-Card (min 16Gb) using Raspberry Pi Images : https://www.raspberrypi.com/software/

- If you want to connect the Raspberry Pi to Wifi : mount the SD-Card on your computer and add a file "wpa_supplicant.conf" to the /boot directory of the mounted volume.
This file should contain the following:

```
network={
ssid="FlyFi"
psk="42805WPA2"
key_mgmt=WPA-PSK
}
```


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
