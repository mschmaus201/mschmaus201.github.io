---
layout: post
title: "Arduino the Ruby Way"
date: 2013-06-17
comments: false
categories: 
---

# Arduino the Ruby Way
#### [Speakerdeck Slideshow](https://speakerdeck.com/austinbv/arduino-the-ruby-way)

###### Ruby in Our World
Over the past two weeks, I've heard a lot about the origin of the Ruby language and Matz's reasoning.  (Paraphrasing) "I didn't want to create a computer language that humans can understand, I wanted to create a human language that computers can understand."  And since beginning work with Ruby, my minimal exposure has opened my eyes to the world of possibilities with this language.  But not just the TRON-like world of software.  I'm talking about our world as well.

There's this little thing out there called an Arduino.  What is that you ask?  I got my buddies at WIRED to right up a little [post](http://www.wired.com/gadgetlab/2008/04/just-what-is-an/) about Arduinos (Arduinoes? Arduini?) for you.  But long story short, its a microcontroller that can be connected with a number of sensors) - including ultrasound, humidity, microphone, etc - that opens the gates and connects our world with the world of TRON.

So, I've got a pop quiz for you.  What's better than the capabilities of an Arduino?  Hint: It starts with an 'r' and rhymes with "doobie."  Its an Arduino powered by Ruby, the "computer language" of our world.

###### Arduino Capabilities
What ~~can~~ can't you do with an Arduino?  Here are a few Arduino projects I've found incredible creative/useful:

1. Using a sensor to detect the moisture of the soil for your plant, a tweet will be sent out as soon as your plant needs to be watered.
2. Make a motion detector for various purposes (including automatically turning on and off lights)
3. If you don't have a thermostat, you can set it to turn on your A/C or heat depending on the indoor and outdoor temperature (indoor using a temperature sensor, and outdoor pulling the info from the internet)
4. Control an RC car using your phone (my personal favorite).

###### Using Ruby with an Arduino
To begin using Ruby to program an Arduino, there will be a few gems to download that will help you out.  There's a link [here](http://playground.arduino.cc/interfacing/ruby) which explains a number of gems.

- To start off, "Dino" will help you get up and running very quickly.  And of course, its an open source gem so its constantly being updated and improved.  "Dino" even creates the class Board to make working with your Arduino even easier!
- "SerialPort" will allow you to read from the Arduino's serial port.  The serial port is the port that allows the Arduino to interact with other devices.
- My personal favorite is the "Arduino" gem which allows you to prototype programs for your Arduino without having to burn them to the device.  That way, you can see on your computer how the program would work without having to deploy it!
- This isn't a gem, but a class which I find fascinating.  The TxRx class allows your Arduino to connect to another Arduino and its serial port (and therefore all the info coming from any attached sensors).  I can't even begin to think the type of network you can build with a few of these things and what you can do with them...


###### Using Ruby with an Arduino
So I've just recently become familiar with the capabilities of an Arduino after another student in my class, Kristen Curtis, posted about it on her [blog](http://picodegallo.github.io/blog/2013/06/10/mentally-reframing-ruby-part-2/).  As I begin to learn more about Ruby, I absolutely plan to implement my new found skills on an Arduino and definitely will keep you posted here!