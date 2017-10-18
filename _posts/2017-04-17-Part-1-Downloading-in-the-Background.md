---
published: false
layout: post
title: Downloading in the Background on a Xamarin.Forms Project - Part 1: Debugging Connectivity
date: 2017-04-17 12:23
comments: true
author: adam
tags: [Xamarin.Forms, Xamarin.iOS, Xamarin.Android, Debugging, Connectivity]
---
# Part 1 - Getting Started: Downloading in the Background on a Xamarin.Forms Project
Recently I was tasked with building the background downloading capabilties for both Xamarin iOS and Xamarin Android on a Xamarin.Forms project.  Internet connectivity has become ubiquitous and users expect downloads to just work.  Getting downloads to "just work" however can be much more difficult than it sounds.

Some users are on WiFi, others are on a great LTE network, while some may be roaming on an old Edge (2G) network or driving through a tunnel or rural area where connectivity comes and goes.  Being able to build a robust sytem that can adapt and inform the user of their connection state is a tall order and **should not** be taken lightly.  

#### Step 1 - Determine Your Audience
If you know your users will always be connected to a WiFi signal in their home/office, build for that.  If your users are primarly on beefy cellular networks build and test for that.  If you have users in other parts of the world where connectivity isn't as reliable you need to know how to test for this scenario as well.

#### Step 2 - Use the Right Tools
We are only as good as our tools! I am a firm believer in finding and using up-to-date effective tools for the job.  So what tools will help build an application that needs to run on a variety of networks?  Here are a couple that I feel make things a lot easier.

1. [Network Link Conditioner](https://developer.apple.com/download/more/?q=Additional%20Tools)
2. [Charles Proxy](https://www.charlesproxy.com)
3. Test on physical devices, not on simulators!

#### Network Link Conditioner
I discovered this tool earlier this year and have been loving it for testing connectivity.  It can be downloaded from the Apple Developer Portal as part of the "Additional Tools for Xcode" package.  After installing the application it can be accessed via the preferences pane in System Preferences on MacOS.  There are many network profiles to play with, such as WiFi, LTE, 3G, DSL, 100% Loss, etc.  The 100% loss profile is awesome because this allows developers to test the scenario where the user may be actively using a fine connection and all of a sudden the connection fails (even though the user is still technically connected to the Internet).  Using this on the Mac is great if you are proxying with a real physical mobile device, however it will have an impact on your Internet usage on the computer while the Network Link Conditioner is turned on.  

A better option (which is my personal preference) is to use the Network Link Conditioner on the physical iOS device itself.  Go to Settings > Developer > Network Link Conditioner Status.  Choose the applicable profile and enable.  Remember to disable this when you are finished profiling the connection because this will impact all other outgoing/incoming connections to the iOS device!  Please see the link to the NSHipster article below that has a great walkthrough on getting started using the Network Link Conditioner.

<img src="https://mobilecomposer.github.io/images/2017-04-17/settings-developer.jpg" />

#### Charles Proxy
Charles is a must have for all developers - it is one of the first apps I purchased when getting into application development!  Charles is a proxy that enables developers to view all HTTP(S) traffic between a machine/device and the Internet.  As I mentioned earlier you can also proxy physical iOS or Android devices with Charles, as well as any traffic on a simulator/emulator.

#### Xamarin.iOS Background Downloading Connectivity Gotchas
There are a few areas around iOS background downloading that I would like to share.
1. iOS Background Transfer Service is completely handled 

#### What's Next
Now that we have determined our audience and have a set of tools to assist in this task, let's dive into downloading in the background on iOS.  Stay tuned for part two!

#### Resources
[Improving HTTP Performance in Xamarin Applications](http://jonathanpeppers.com/Blog/improving-http-performance-in-xamarin-applications)
[Network Link Conditioner](http://nshipster.com/network-link-conditioner/)
[Walkthrough - Using Background Transfer Service and NSURLSession](https://developer.xamarin.com/guides/ios/application_fundamentals/backgrounding/part_4_ios_backgrounding_walkthroughs/background_transfer_walkthrough/)
[Charles Proxy](https://www.charlesproxy.com)
