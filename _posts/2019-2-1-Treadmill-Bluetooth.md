---
layout: post
title: Hacking the bluetooth of my treadmill
published: true
---

In order to keep exercising my runs during the dark and cold winter months, I decided to buy a treadmill to be able to train at all weather conditions. Of course running on a treadmill can be quite boring so I was looking into gathering the data from the treadmill and using it to create my own custom dashboards and to connect it to Strava, the Social Network for Athletes.

The treadmill I decided to buy was the "**Focus Fitness Senator**".

![Focus_Fitness_Senator.jpg]({{site.baseurl}}/images/Focus_Fitness_Senator.jpg)

It’s a decent device with a powerful engine of 3Pk and a bigger sized belt. 
You can connect it to your smartphone or tablet via bluetooth but the mobile app, called “e-Health”, only allows to control the treadmill remotely and store run sessions, but there’s no way to download that data. Initially I tought I would also be able to connect the treadmill to Zwift and take it for a virtual run but unfortunately my treadmill is not supported by Zwift.

Ok, so let’s dig a bit deeper into the bluetooth connection and the services that the treadmill exposes. First we can have a look at the details of the bluetooth service that the treadmill is broadcasting. For that we can use a mobile App called “**nRF Connect**” from Nordic Semiconductor.
I recommend using the App on an Android device, as it has more features than the one on iPhone.
Additionally the android device allows to log all bluetooth traffic which we will need for further analysis later on.

![Bluetooth_Service.png]({{site.baseurl}}/images/Bluetooth_Service.png)

The “nRF Connect” shows that the treadmill exposes an “Unknown Service” which consists of two Characteristics.
The first one with UUID “0000fff1-0000-1000-8000-00805f9b34fb” will allow us to be notified when new values come in.
Now how do we enable these notifications ? Simply by registering to this characteristics.

More details on these bluetooth services and characteristics can be found in the ESP bluetooth experiments from Andreas Spiess (the guy with the Swiss accent) - [https://www.youtube.com/watch?v=osneajf7Xkg&t=530s](https://www.youtube.com/watch?v=osneajf7Xkg&t=530s)

## Debugging bluetooth communication

As the activation of the notification flag didn’t give me any values back from the treadmill, I decided to debug the bluetooth conversation from within the mobile App "e-Health" that connects to the treadmill.
For this I had to activate bluetooth logging from within the developer options on my Android phone.

![Android_Dev_Options.png]({{site.baseurl}}/images/Android_Dev_Options.png)

I started the app on my Android phone, connected to the treadmill, started the belt and then stopped it. That’s already enough data to start the analysis.

All we have to do now is take the bluetooth trace file from the phone and load it up into “Wireshark” - the most widely-used network protocol analyzer. 

In order to get this file from your phone, you need to have the Android tools installed on your computer. You can then use the **“adb pull /sdcard/btsnoop_hci.log”** command to retrieve the file. 

First look at the Wireshark bluetooth conversation data seems overwhelming and I can tell you I spend quite some time analyzing the flow of data. The data we’re actually interested in is the bytes that are send by the treadmill. 
This can be done by applying a filter **“bthci_acl_dst.name == “EW-TM-0767”** (EW-TM-0767 is the bluetooth name of my treadmill). Now we can see all the write commands that are send out. 

Looking at the data, I noticed that a specific byte-stream is re-occuring : **“Value f0c303000000b6”**.

![Wireshark_bluetooth.png]({{site.baseurl}}/images/Wireshark_bluetooth.png)

Ok, so let’s try to write that value into the characteristic of our tool ”nRF Connect”… and bingo !
Now we get a value back within our first bluetooth service characteristic.

![Bluetooth_Write.png]({{site.baseurl}}/images/Bluetooth_Write.png)

After some testing with different speeds and elevations on the treadmill I discovered the meanings of the different byte responses:

![Bluetooth_bytestream.png]({{site.baseurl}}/images/Bluetooth_bytestream.png)

It's not complete but the most important data fields are now revealed so we can take it a step further.

In a next post I will explain howto connect an ESP32 to my treadmill and send the data to the IBM Watson IoT Platform Service.
