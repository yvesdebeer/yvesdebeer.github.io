---
published: true
---
In my previous blog, I explained howto to get access to the the bluetooth service provided by the treadmill.

Now we take it a step further, and use an ESP32 microcontroller to connect to the treadmill and process the data during a running activity. I will be using an "Adafruit HUZZAH32 ESP32" for this.

![Adafruit-HUZZAH32]({{site.baseurl}}/images/Adafruit-HUZZAH32.jpg)

**Now, howto get started with programming the ESP32 using Arduino ?**

I was insprired by the ESP bluetooth experiments from Andreas Spiess - 
[https://www.youtube.com/watch?v=osneajf7Xkg&t=530s](https://www.youtube.com/watch?v=osneajf7Xkg&t=530s)

You can also find bluetooth example sketches within the Arduino examples library. 
My code is based on the example "BLE Client" and you can find the code here : [Treadmill_BLE_Client.ino](https://github.com/yvesdebeer/Treadmill-Bluetooth-IoT/blob/master/Treadmill_BLE_client/Treadmill_BLE_client.ino)

Once you upload the sketch to the board it will first run through setup().
This will activate bluetooth and start to scan for services. Once we find our service, the scan will be stopped and the "connectToServer" function will initiate a connection, check the existance of our service definition and try to obtain a reference to the bluetooth characteristics. With our first characteristic we will also check the notification flag and by registering the callback function, the notification will automatically be enabled.

![Bluetooth_Service.png]({{site.baseurl}}/images/Bluetooth_Service.png)

The Arduino main loop() function will update our second characteristic with a value "f0c303000000b6" which will trigger the treadmill to send a notification and a response value within our first characteristic.
This mysterious value was obtained via bluetooth hacking as explained in my previous blog.

As shown in the Arduino monitor we are getting new values from treadmill every second. Details on the response value are shown below:

![Bluetooth_bytestream.png]({{site.baseurl}}/images/Bluetooth_bytestream.png)

Output from the Arduino serial monitor:

	Starting Arduino BLE Client application...
    BLE Advertised Device found: Name: EW-TM-0767
    Forming a connection to 68:9e:19:16:2a:0f
     - Created client
     - Connected to server
     - Found our service
     - Found our first characteristic0000fff1-0000-1000-8000-00805f9b34fb
    Setting notifyCallback
     - Found our second characteristic0000fff2-0000-1000-8000-00805f9b34fb
    Trying to write
    We are now connected to the BLE Server.
    Notify callback for characteristic 0000fff1-0000-1000-8000-00805f9b34fb of data length 20
    data: F0D3100000000000010000D4
    Notify callback for characteristic 0000fff1-0000-1000-8000-00805f9b34fb of data length 20
    data: F0D3100000000000010000D4


In a next blog, I will explain howto setup the IBM IoT Platform service and howto adapt the arduino code in order to send the treadmill responses in real time via MQTT.
