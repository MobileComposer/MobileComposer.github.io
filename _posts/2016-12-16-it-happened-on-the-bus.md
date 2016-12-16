---
published: true
layout: post
title: It Happened on the Bus
---
Over the last few weeks we've noticed that a **lot** happens in your UWP project when you flip the switch from Debug build to Release build mode.  At the heart of it, has been this little check box on the UWP project build properties: "Compile with .NET Native tool chain".  There are some good posts about [what it means for UWP.]( https://blogs.windows.com/buildingapps/2015/08/20/net-native-what-it-means-for-universal-windows-platform-uwp-developers/#HG2ld3KGHUOiMVQI.97)  [Here also.](http://stackoverflow.com/questions/37759125/windows-store-apps-windows-8-vs-uwp)

For me, it means that our app can make the long journey into the Windows Store and none of the code-drawn graphics will be displayed on screen.  **Oops**.

We've been using the NControl library in this app for a while and it's been a great tool for drawing custom vector graphics via code.  [NControl](https://github.com/chrfalch/NControl) is a simple Xamarin.Forms wrapper control around the library that does the drawing, NGraphics.  
[NGraphics](https://github.com/praeclarum/NGraphics) is a cross platform library for rendering vector graphics on .NET.  It provides a unified API for both immediate (display to screen) and retained mode (save .png file to disk) graphics using high quality native renderers.

Here's a little sample of how to draw something:

```csharp
var width = 200;
var height = 200;

var frame = new NGraphics.Rect(-width/2, 0, width, height);
var corner = new NGraphics.Size(1, 1);
var pen = new NGraphics.Pen(NGraphics.Colors.Blue, 0);

var point1 = new NGraphics.Point(0, 0); // these don't seem to have any effect on the gradient
var point2 = new NGraphics.Point(0, 0);
var brush = new NGraphics.LinearGradientBrush(point1, point2, NGraphics.Colors.Blue, NGraphics.Colors.Green);

var control = new NControlView
{
    DrawingFunction = (canvas, rect) =>
    {
        canvas.DrawRectangle(frame, corner, pen, brush);
    },
    HorizontalOptions = LayoutOptions.Center
};
```

And this is the result  
<img src="{{site.baseurl}}/images/2016-12-16/Screen Shot 2016-12-16 at 4.01.43 PM.png" style="width: 200px;"/> 

(Note that the docs on the GitHub page are a bit out of date)

Impressively, NControl currently supports native custom renderers for 6 platforms: iOS, Android, Windows Phone (8, 8.1, and Silverlight 8.1), and Windows Store (Windows 8.1), but was lacking support for UWP (Windows 10).  I built this out about 6 months ago and it's been working really well in our project since.  Imagine my surprise when it simply didn't work at all in release mode :/  When adding UWP support for these libraries, I used a common "monkey-see, monkey-do" approach, not knowing much of the graphic-y bits in the libraries.  So naturally, I assumed that the monkey messed something up along the way.

Here's a comparison of the Live Visual Tree:   
<table>
<tr>
<td>Here without .NET Native (Release build) enabled</td>
<td>And with it enabled</td>
</tr>
<tr>
<td style="vertical-align: top">
<img src="{{site.baseurl}}/images/2016-12-16/Live Visual Tree.png" alt="NControl is in Visual Tree" style="width: 350px;"/> 
</td>

<td style="vertical-align: top">
<img src="{{site.baseurl}}/images/2016-12-16/Live Visual Tree - No NControl.png" alt="NControl is gone" style="width: 375px;"/>
</td>
</tr>
</table>
  
  
# Debugging .NET Native

Debugging the issue has been a process of examining each component and revisiting the code I wrote, starting at the lowest layer, NGraphics.  The good news there is that it works fine when compiled with .NET Native.  Sweet!  This also gave me the chance to improve things a bit and submit a pull request back to the library (more on that below). 

On to NControl then - it contains all the necessary Windows-specific stuff, like a Windows.UI.Xaml.Controls.Canvas, and helpers to convert the NGraphics.Brush to a Windows.UI.Xaml.Media.Brush, but most importantly, a Custom Renderer that renders an NControlView as a Windows.UI.Xaml.Controls.Grid that can be displayed on UWP.  Hooking the NControl source to my repro project seemed to be the best approach.  As you may have guessed, debugging a .NET Native compiled app is a little different.  Check out [this post on MSDN](https://blogs.msdn.microsoft.com/visualstudioalm/2015/07/29/debugging-net-native-windows-universal-apps/) for directions.

This confirmed the true issue; the constructor of my NControlViewRenderer was never getting hit, thus all the great code I had added for UWP wasn't even getting touched :/  
I've seen this happen before when I forgot to add the assembly attribute above my renderer class.
```csharp
[assembly: ExportRenderer(typeof(NControlView), typeof(NControlViewRenderer))]
```
but that wasn't the issue here.  It's almost like Xamarin.Forms was totally oblivious of this custom renderer...  But why?  Why?  
Why...  
So I searched all the places I usually look: Google, Xamarin Forums/Bugzilla/Docs (_is that the best order?_), pdf copies of books, GitHub issues.  Zero.

It was then that I hit the point that I often hit when troubleshooting development issues

 **_This problem can't be this huge._**

If no 3rd party library's custom renderers work in Xamarin.Forms apps in the Windows Store, that would be a HUGE deal.  It can't be that hundreds of Xamarin developers around the globe are having this issue, can it?  It'd have turned up in a search somewhere, right?  Someone has got to be writing UWP apps for the Windows Store, right?  (maybe not...)


# Leave Curious

So I left work curious (I'd recommend it), and decided to Google just a bit on the bus ride home.  I think the small keyboard on my phone encouraged me to keep the search terse; just the essential keywords of my problem: **UWP ".NET Native" ViewRenderer**

And then I found it.  

One line in an unrelated library's FAQ section about license keys: 

> "For .NET Native compilation, you have to tell Xamarin.Forms which assemblies it should scan for custom controls and renderers"

AH!  That makes so much sense!

Now there have been a few UWP-specific Xamarin.Forms ah... _intricacies_ like this (I'll post on them later).  Some aren't documented at all, and some are, but not very well, like this one.  There is a special overload of the Forms.Init method just for UWP in which you can pass in an IEnumerable called rendererAssemblies:

```csharp
#if WINDOWS_UWP
    public static void Init(IActivatedEventArgs launchActivatedEventArgs, IEnumerable<Assembly> rendererAssemblies = null)
#else
```

(BTW, you can check it out for yourself in the [offical Xamarin.Forms repo here)](https://github.com/xamarin/Xamarin.Forms/blob/master/Xamarin.Forms.Platform.WinRT.Tablet/Forms.cs#L28)

And what does it do with these assemblies?

```csharp
#if WINDOWS_UWP
    Registrar.ExtraAssemblies = rendererAssemblies?.ToArray();
#endif

Registrar.RegisterAll(new[] { typeof(ExportRendererAttribute), typeof(ExportCellAttribute), typeof(ExportImageSourceHandlerAttribute) });
```

And why would you want to register these extra assemblies?

> "Xamarin.Forms may be unable to load objects from those assemblies (such as custom renderers)."  

But the [little snippet of "documentation"](https://developer.xamarin.com/guides/xamarin-forms/platform-features/windows/installation/universal/#Troubleshooting) is a little murky here - there are directions to use this UWP-specific overload, but it's described as a fix for the "Target Invocation Exception" when using "Compile with .NET Native tool chain".  I wasn't seeing any exceptions, so the entire section under this heading wasn't on my radar at all.  The app ran great.  Blissfully unaware that my NControlView should be displayed on the screen.

In fact, I found that this is a requirement for ALL custom renderers that live in a 3rd party library.  For example, our app has a custom renderer for playing videos.  It has an implementation for each platform (iOS & UWP) that lives in its respective platform-specific projects, and this renderer works great.  However, in addition to NControl, we're also using the popular Xamarin Image Circle Control plugin and it isn't working correctly in .NET Native builds, leaving the images square.  Same issue.

So what other assemblies do we have to include?  
You may notice if you look at the Xamarin Evolve [source code](https://github.com/xamarinhq/app-evolve/blob/master/src/XamarinEvolve.UWP/App.xaml.cs#L92) that James seems to add everything and the kitchen sink here. Not sure if all that is needed.  
The only answer I've heard from Xamarin was from James: "_Mostly you just need third party ones. I just did them all lol cause I wasn’t sure to be honest_"


# The Fix

6 days in the making, here's the bit of code that corrected it:

```csharp
// For .NET Native compilation, you have to tell Xamarin.Forms which assemblies it should scan for custom controls and renderers
var rendererAssemblies = new[]
{
    typeof(NControl.UWP.NControlViewRenderer).GetTypeInfo().Assembly,
    typeof(ImageCircle.Forms.Plugin.UWP.ImageCircleRenderer).GetTypeInfo().Assembly
};
	
// Then call Init with these assembiles
Xamarin.Forms.Forms.Init(e, rendererAssemblies);
```

Even though I was calling the Init methods on these libraries:

```csharp
ImageCircleRenderer.Init();
NControl.UWP.NControlViewRenderer.Init();
```    

Without the above code, you get nothing in UWP with .NET Native builds.  No exceptions are thrown.  Nothing in the debug output window.  Nothing on screen.  Silent failure.


**I feel that this should be documented better.**  Hence this blog post.


If you are interested in using NControl or NGraphics on UWP, check out the pull requests [here](https://github.com/praeclarum/NGraphics/pull/63) and [here](https://github.com/chrfalch/NControl/pull/71).  

I'd love it if you would  
1.) try it out  
2.) give any feedback on how it could be improved.  
I'm really excited about this, my first PR to an open source library, so let's work together people!

--> Tom
