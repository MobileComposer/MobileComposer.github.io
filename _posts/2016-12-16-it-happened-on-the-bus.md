---
published: false
layout: post
title: It Happened on the Bus
---
A story starting with that line never dissapoints.

Over the last few weeks we've noticed that a lot happens in your UWP project when you flip the switch from Debug build to Release build mode.  At the heart of it, has been this little check box on the UWP project build properties: "Compile with .NET Native tool chain".  There are some good posts about [what it means for UWP.]( https://blogs.windows.com/buildingapps/2015/08/20/net-native-what-it-means-for-universal-windows-platform-uwp-developers/#HG2ld3KGHUOiMVQI.97) [Here also.](http://stackoverflow.com/questions/37759125/windows-store-apps-windows-8-vs-uwp)

For me, it means that our app can make the long journey into the Windows Store and none of the code-drawn graphics will be displayed on screen.  **Oops**.

We've been using the NControl library in this app for a while and it's been a great tool for drawing custom vector graphics via code.  [NControl](https://github.com/chrfalch/NControl) is a simple Xamarin.Forms wrapper control around the library that does the drawing, NGraphics.  [NGraphics](https://github.com/praeclarum/NGraphics) is a cross platform library for rendering vector graphics on .NET.  It provides a unified API for both immediate (display to screen) and retained mode (save .png file to disk) graphics using high quality native renderers.

[show code example and image. Note that the docs are a little out of date.]


Impressively, NControl currently supports native custom renderers for 6 platforms: iOS, Android, Windows Phone (8, 8.1, and Silverlight 8.1), and Windows Store (Windows 8.1), but was lacking support for UWP (Windows 10).  I built this out about 6 months ago and it's been working really well in our project since.  Imagine my surprise when it simply didn't work at all in release mode **:/**  When adding UWP support for these libraries, I used a common "monkey-see, monkey-do" approach, not knowing much of the graphic-y bits in the libraries.  So naturally, I assumed that the monkey messed something up along the way.
  
  
## Debugging .NET Native

Debugging the issue has been a process of examining each component and revisiting the code I wrote, starting at the lowest layer, NGraphics.  The good news there is that it works fine when compiled with .NET Native.  Sweet!  This also gave me the chance to improve things a bit and submit a pull request back to the library (more on that below). 
On to NControl then - it contains all the necessary Windows-specific stuff, like a Windows.UI.Xaml.Controls.Canvas, and helpers to convert the NGraphics.Brush to a Windows.UI.Xaml.Media.Brush, but most importantly, a Custom Renderer that renders an NControlView as a Windows.UI.Xaml.Controls.Grid that can be displayed on UWP.  Hooking the NControl source to my repro project seemed to be the best approach.  As you may have guessed, debugging a .NET Native compiled app is a little different.  Check out this post for directions. [link to post: https://blogs.msdn.microsoft.com/visualstudioalm/2015/07/29/debugging-net-native-windows-universal-apps/]
This confirmed the true issue; the constructor of my NControlViewRenderer was never getting hit, thus all the great code I had added for UWP wasn't even getting touched :/  I've seen this happen before when I forgot to add the assembly attribute above my renderer class: [assembly: ExportRenderer(typeof(NControlView), typeof(NControlViewRenderer))] - but that wasn't the issue here.  It's almost like Xamarin.Forms was oblivious of this custom renderer...  But why?  Why?  So I searched all the places I usually look: Google, Xamarin Forums/Bugzilla/Docs (is that the best order?), pdf copies of books, GitHub issues.  Zero.

It was then that I hit the point that I often hit when troubleshooting development issues: This problem can't be this huge.  If no 3rd party library's custom renderers work in Xamarin.Forms apps in the Windows Store, that would be a HUGE deal.  It can't be that hundreds of Xamarin developers around the globe are having this issue, can it?  It'd have turned up in a search somewhere, right?  Someone has got to be writing UWP apps for the Windows Store, right?  (maybe not....)


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

