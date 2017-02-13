---
published: true
layout: post
title: Wired Debugging on a Surface Tablet? Yes You Can!
author: tom
date: 2017-02-10 12:00
tags: [UWP, Surface, Debugging, Xamarin, Xamarin.Forms]
---

<div>
  <div style="display: inline-block;" align="top">
    <img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/SurfacePro3USBPort.png" width="150" />  
  </div>
  <div style="display: inline-block; width: 80%; vertical-align: top;">From a mobile developer's perspective, debugging on a Surface tablet is weird. Really weird. 
  The first time I picked up the Surface and spotted the USB port on the side, I figured, cool - I'll just plug it into my laptop and run something! <br/>
  Ah.... no. <br/>
  Sorry. <br/>
  It's not <i>that</i> kind of USB port.  You can't just stick anything in there, man.  
  </div>
</div>

To debug on a Surface (or devices that do not have Visual Studio installed) you must install the Visual Studio Remote Debugger client app on the device and then wirelessly communicate with it to debug from Visual Studio on your laptop.  The version of the remote client app MUST match the version of Visual Studio you're using (update 1, 2, 3, etc.).  Directions to do that is [here](https://msdn.microsoft.com/en-us/library/y7f5zaaa.aspx)  
<img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/RemoteDebuggerClient.png" width="400" />  

<br />
There is a **major** issue with this plan, however:  
More and more apps we're building need to be offline capable.  So how are you supposed to debug in an offline scenario with wireless debugging?  That's where that USB port is going to come in handy.  

Also, another benefit of debugging over a wired connection is that it's a bit more reliable and faster to get the app running after each build.

One thing to note: like a lot of mobile developers, I work all day on a MacBook Pro and use [Parallels](http://www.parallels.com) that runs a Windows 10 Virtual Machine so I can use all my Windows tools side by side - so this post is written from that perspective.

# Hardware Needed

In order to debug over a wired connection, you'll need a few things:

 - 2 USB to Ethernet dongles.  You can find them for pretty cheap on [Amazon](https://www.amazon.com/gp/product/B00ET4KHJ2/ref=oh_aui_detailpage_o08_s00?ie=UTF8&psc=1).
 - A length of cat 5 cable to connect the two dongles together.

<br />

What won't work here is one of those fancy (and 3x more expensive) lightning to Ethernet dongles plugged into my Mac, because my Windows VM doesn't recognize it.  The USB adapter is the one Windows understands.  
<img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/thunderboldToEthernetAdapter.png" width="250" />  

<div>
  <div style="display: inline-block; width: 50%; vertical-align: top;" align="top">
    Connect the dongles together with the ethernet cable, and plug one dongle into your laptop and the other into the Surface. <br/>
    Here's what it should look like when it's all hooked up.
  </div>
  <div style="display: inline-block;">
    <img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/WiredSetup.jpg" width="450px" />
  </div>
</div>

# Debugging Online and Offline

For our Xamarin.Forms app, we need to be online the first time a user logs in and syncs data with the server, but then we want to disconnect from the internet to test offline scenarios after that first login. 

The trickiest part was getting my laptop to share its internet connection with the Surface over my wired ethernet connection.  When you first connect your Surface to your laptop, if you turn off the Surface WiFi connection you'll see an icon indicating that there is no internet connection.  

Thankfully, there's a nice Windows feature that allows you to do that called Adapter Bridging.  
Here's how to do it:

1. Open **Network Connections** on Windows
2. Locate the adapter that is supplying your internet connection. With Parallels this is done by an Ethernet adapter
3. Select both the adapter from step #2 and the one for your plugged in USB dongle
4. Right click and select **Bridge Connections**  
<img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/BridgeConnections.png" width="400px" />  
The internet connection will now be shared over the ethernet cable to your Surface device  



**Start debugging while online:**  

1. Bridge adapters  
  OR  
  You can always simply unplug the USB adapter dongle from the Surface and turn WiFi back on.
2. Open browser on Surface to check for connectivity
3. Get the Surface's IP address (cmd > ipconfig)
4. Enter the IP address in the Remote Machine textbox in Visual Studio.  Format is xxx.xxx.xxx.xxx:[Port Number]
5. Run on Remote device

**Then offline:**  

1. Select adapters, right click, select Remove from Bridge 
2. Wait for Surface to get new IP
3. Type it in the Remote Machine textbox in VS



# Tips

- When Un-Bridging.  The Surface will show that the internet connection is lost right away and long before it gets a new IP, so wait for the USB Ethernet adapter to show that it has a new (different) IP before trying to debug to it in Visual Studio.  

- Generally, you'll have your Solution build configuration set to deploy the UWP head project to your device each time you start debugging the app. That means the app will be installed over the old one on each deploy and that can be undesirable when testing offline behavior, so don't forget to uncheck that if needed after the initial deploy of the app to the device.  
<img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/SolutionConfig.png" width="800px" />  

- I always turn on these two options in the VS Remote Debugger app running on the Surface (under **Tools** menu > **Options**):
  1. **No Authentication**  
  Turning this off seems to alleviate a lot of the hassle of trying to get the debugger to connect to the remote client app.  I debug on a private or wired network and only have the remote client running when I need to debug, so the lack of security doesn't concern me here.
  2. **Allow any user to debug**  
  I use this setting because don't log into my Windows 10 VM via Parallels so I've had an issue with that.  I also use this when my coworker needs to debug on the Surface.  
  <img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/AuthMode.png" width="300px" />  
  
- If you find yourself switching from connected in disconnected frequently, it's a pain to keep typing in the same 2 IP addresses over and over in the Remote Machine textbox. Undo (Ctrl + Z) works inside that textbox.



# Guiding Principles

 - Don't live or die by what the **Remote Connections** dialog is able/unable to find. It's nice when it finds your device, but often it doesn't.  So if you've got things hooked up correctly, just type the IP + Port in the address box manually.  Remember to use the correct format: ip**.**addr**.**goes**.**here**:**[port number]  
 <img src="{{site.baseurl}}/images/2017-02-10/WiredSurfaceDebugging/ConnectionFound.png" width="350px" />

 - You must have the Visual Studio 2015 Remote Debugger app running on the Surface at all times. That app will close automatically after a certain period of inactivity.
 


# Common Errors

**Error:** Unable to connect to the Microsoft Visual Studio Remote Debugger named '10.211.55.5:4020'.  The Visual Studio Remote Debugger on the remote computer is running as a different user.

**Fix:** On the Surface, in the menu for the VS 2015 Remote Debugger app, go to Tools > Options, and check the **Allow any user to debug** option, under the **No Authentication** radio button.


# Resources
- [https://msdn.microsoft.com/en-us/windows/uwp/debug-test-perf/deploying-and-debugging-uwp-apps](https://msdn.microsoft.com/en-us/windows/uwp/debug-test-perf/deploying-and-debugging-uwp-apps)
