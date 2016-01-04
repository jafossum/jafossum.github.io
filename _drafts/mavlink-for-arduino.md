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

At the start of my own Arduino based Quad Drone project at home, I was looking for the best way to communicate
between my remote and my Arduino based drone. In one of my earlier posts I gave a short description about using the PS3 SixAxis controller instead of a RC Remote,
and this is also the plan for my Quad.

To be able to easy support and integrate functionality on a later stage of development, I was looking around the great web for solutions on this type of communication.
and there it was: [MAVLink (Micro Air Vehicle Communication Protocol)](http://qgroundcontrol.org/mavlink/start).

MAVlink is a light weight message marshaling library first developed back in 2009 by [Lorenz Meier](http://www.inf.ethz.ch/personal/lomeier) for use in micro air vehicles.

MAVLink has a set of predefined messages and structures for use between a "ground station" and the air vehicle.
There are a few pre-generated data sets and message structures to be used, or it is possible to make your own by modifying or generating a set of XML files in the library.
A description on how this can be done is found [here](http://qgroundcontrol.org/dev/mavlink_onboard_integration_tutorial),
or more Arduino specific [here](http://qgroundcontrol.org/dev/mavlink_arduino_integration_tutorial).

## Generate MAVLink Headers

To get started with the integration of MAVLink, you first need to get the correct .h (Header) files to use. There are some predefined headers
generated from the last stable release on [GitHub](https://github.com/mavlink/c_library) for direct download and ease of use.

I wanted to use my own set of supported messages, so I decided to use the [MAVLink Generator](https://github.com/mavlink/mavlink)
to be able to make my own set of supported messages. For simplicity I will be using the same example as posted on the MAVLink homepage,
integrating the HeartBeat message, but under my own MAVLink protocol alias *JAF_QuadProject*.

# MAVLink XMLs

First of all there are no code to be written when to want to generate new messages, just .xml
files that are used by the mavgenerate.py python script provided by the [MAVLink repository](https://github.com/mavlink/mavlink).
Details on this can be found [here](http://qgroundcontrol.org/mavlink/create_new_mavlink_message).

There is a .xml file allready created in the MVLink project that supports only the HeartBeat functionality. This is
called minimal.xml. Copying the contents of minimal.xml into JAF_QuadProject.xml I have made the necessary preparations to get started.

After running mavgenerate.py directly from the project root folder i get a GUI with simple options.
	- Choose File
	- Output Destination
	- Language
