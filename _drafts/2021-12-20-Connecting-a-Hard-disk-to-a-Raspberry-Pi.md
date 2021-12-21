---
published: false
---
A Raspberry Pi can offer a quick solution to do some prototyping but if you need additional storage, an external hard drive can be very handy.

I was looking for a tool to store security camera feeds using a Raspberry Pi.
Even if you're just storing footage for a couple of days, an SD-Card is not a good option.
It's not so fast and it could fail overtime due to lots of write operations.

Following steps describe how to connect an external USB Hard Drive/SSD to a Raspberry Pi.
I'm using an old Raspberry Pi-3 which unfortunately does not support booting from a USB Drive.
So I still need to boot the Pi using a small SD Card (2Gb works just fine).

Here are the steps I took:

- First prepare an SD-Card (min 16Gb) using the '[Raspberry Pi Imager](https://www.raspberrypi.com/software/)'-tool.

- Next steps require an external screen (connected via HDMI), an external keyboard and a mouse :

Boot the Pi and connect your external USB drive. I'm using an SSD drive because it's faster and doesn't need an additional power supply.

- From the menu : goto 'Acccesories' -> 'SD Card Copier'
- Select the source and target devices. Make sure to enable 'New PArtition UUIDs':
  This will copy the 2 partitions of the SD-Card onto the external USB drive.

   ![PI_CP_SD.png]({{site.baseurl}}/images/PI_CP_SD.png)

- Reboot the Pi. This will automatically mount the external drive.
- Note down the PartitionUUID from the file cmdline.txt from the /boot directory of the external disk. This is the UUID we want to mount when the Pi boots up.
- Now edit the cmdline.txt from the /boot directory and replace the root partition UUID to refer to the UUID from the external disk e.g.

		console=serial0,115200 console=tty1 root=PARTUUID=a4cd54ec-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles
    
    


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
