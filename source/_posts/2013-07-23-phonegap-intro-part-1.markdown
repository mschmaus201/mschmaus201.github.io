---
layout: post
title: "PhoneGap: Intro (Part 1)"
date: 2013-07-23 00:11
comments: true
categories: 
---

This is the first of two posts relating to PhoneGap.  This post will be my introduction to PhoneGap, and the second post will be a walkthrough of building my first PhoneGap-based app.  

### What is PhoneGap?  
PhoneGap is an open source framework used to allow you to easily build cross-platform mobile apps using HTML, CSS, and JavaScript.  More so, it allows you to natively interact with the phone, meaning it gives you access to a phone’s features including the accelerometer, GPS, compass, notification center, etc.  Without PhoneGap, a developer would most likely use Java and Objective-C to develop natively for Android and iOS, respectively.  

### History  
To start off, its fascinating to go back into PhoneGap’s archives and read their blogs from back in the day when they first announced and released information on their framework!  I definitely recommend taking a look [here](http://phonegap.com/blog/).   

PhoneGap was first announced in 2008 for the purpose of allowing native mobile app features available to mobile web apps, taking the logic of Adobe AIR, which allows web developers to build Windows and OSX applications.  

### Set Up
PhoneGap is built on open source software called Apache Cordova.  To build something with PhoneGap, you’ll be using the cordova command-line interface.  

First step is to install Node.js  
<img src="/images/node_js_installation.png">  

Then, you must install cordova using: *sudo npm install -g cordova* in the terminal.  "npm" stands for node package manager,  which, if you could guess, is the packet manager for the Node JavaScript platform.  This is installed when you download Node.js.  

To create a new project directory, you need to run *cordova create DirectoryName*.  This will create three subdirectories, including: *merges*, *plugins*, and *www*.    
<img src="/images/cordova_phonegap_directory.png">

*platforms*, the additional directory pictured above is for the target platform(s) you'd like to develop for - ie, iOS, Android, etc.  

All subdirectories are empty except for the *www* directory, of which you can see the content in the image above. The *www* directory is where the application's home page lives.  And for each platform you create (I'll get into this in Part 2 of the post, but the command to add a platform (iOS for exampke) is *cordova platform add ios*), another *www* directory is created within the platform/ios/ directory path.

For my follow up post, I'll install the iOS SDK and create my first PhoneGap app!  Stay tuned.