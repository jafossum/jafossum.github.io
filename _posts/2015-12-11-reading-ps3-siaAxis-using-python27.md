---
layout: post
title: Reading PS3 SixAxis using Python 2.7
description: "Reading controllerdata from the PS3 SixAxis controller"
modified: 2015-12-11
tags: [PS3, Remote, SixAxis, Python, Linux, Arduino, RaspberryPi]
#image:
#  feature: abstract-3.jpg
#  credit: dargadgetz
#  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

I have been thinking about a remote controlled drone project for quite some time.
Together with this I also have a small Arduino controlled car which I bought to get
started from [DealExtreme](http://www.dx.com/).

The problem with these remote controlled projects has always been getting a good
remote control! My plan has always been **NOT GETTING** a RC remote.
So there must be a different way!

How about using my PS3 SixAxis controller for this?

<figure>
	<img src="/images/PlayStation3-Sixaxis.png" alt="">
	<center> <figcaption>PlayStation PS3 SixAxis Controller</figcaption></center>
</figure>

I tried different approaches using bluetooth and different strange program packages downloaded
from different places, but none of these seemed to work.

### Linux Devices

I have some experience in using python 2.7 on a linux platform, so I started out trying to
find the actual device for the PS3 controller. And after connecting the USB cable to my
computer it showed up as the following:

```
$ /dev/input/js0
```

Or even better: Using the symlink so that the device number does not matter.

```
$ /dev/input/by-id/usb-Sony_PLAYSTATION_R_3_Controller-joystick
```

This will work on most Linux systems, and has been tested on Ubuntu and RasperyPi (Raspbian)

### Reading binary data from the device

Python uses the binascii library to binary code and decode text. So using this to read from the device actually makes the data understandable.

{% highlight python %}
from binascii import hexlify
{% endhighlight %}

#### Decoding the data

After finding a similar example on [GitHub](www.github.com) by [@urbanzim](https://github.com/urbanzrim/ps3controller)
i found that each message from the PS3 SixAxis controller is a 8 char long message with different types of data
with a set definition for each of these characters.

Some of these characters are incrementing counters and things I did not care about at first,
but i found that char 8 (string\[7]) has the channel identifier, and that
char 6 (string\[5]) has the actual analog signal value from that channel.

After some research I got this list of analog signals amd digital signals:

| Signal     | Reference | Analog   | Binary |
| :--------- | :-------- | :------- | :----- |
| 00 | L-Stick LR | A | L3 |
| 01 | LeftStick UD | A | Select
| 02 | RightStisk LR | A | R3
| 03 | RightStick UD | A | Start
| 04 | Up Pad | | B |
| 05 | Right Pad | | B |
| 06 | Down Pad | | B |
| 07 | Left Pad | | B |
| 08 | Up Pad | A | L2 |
| 09 | Right Pad | A | R2 |
| 0a | Down Pad | A | L1 |
| 0b | - | - | R1 |
| 0c | L2 | A | - |
| 0d | R2 | A | - |
| 0e | L1 | A | - |
| 0f | R1 | A | - |
| 10 | Triangle | A | - |
| 11 | Rond | A | - |
| 12 | Cross | A | - |
| 13 | Square | A | - |
| 17 | Rotation Axis ? | A | - |
| 18 | Rotation Axis ? | A | - |
| 19 | Rotation Axis ? | A | - |
| 1a | Rotation Axis ? | A | - |

This might seem a bit confusing, and it was but the same identifier was used for different buttons
(binary and analog). In my python approach i focus only on the analog datastream,
and filter out the digital (0 value) digital signals.

#### Putting the data to use

When looking at the data streaming from the PS3 controller,
it is obvious that these are event based, and are only updated when there is a change to any one of the sensors.
Because of this the datastream will be asynchronous.

##### Simple usage

When reading from any file or device it is importsnt that you are able to
cleanly close the file or device after use. python has a nice solution to this using the keyword *with()*

{% highlight python %}
path = "/dev/input/by-id/usb-Sony_PLAYSTATION_R_3_Controller-joystick"
with open(path, 'r') as filestream:

    signals = [0]*27

    while True:
        start = time()
        read_data = f.read(1)
        sdata += [hexlify(read_data)]

    if len(sdata) == 8:

        # Analog button signals
        if (int(sdata[7], 16) > 3) and (int(sdata[5], 16) != 0):
            signals[int(sdata[7], 16)] = int(sdata[5], 16) ^ 0x80
        
        # Stick Signals
        elif(int(sdata[7], 16) <= 3):
            signals[int(sdata[7], 16)] = (int(sdata[5], 16) ^ 0x80) - 128

    print signals
{% endhighlight %}

The resulting printout of this simple function will be a 27 char long list of analog values
according to the table shown above. (All values have been normalized so that center stick,
or no button press will have the default value 0).

The List values will only be updated when the actual signal is changing
from the PS3 remote itself (All signals show up on first run, so the list will allways be complete
after initiating).

This is still a work in progress, but I will make a python package for this in the future and host
it on [GitHub](https://github.com/jafossum/)