---
published: true
layout: post
title: Easy Access to Image and Object Width & Heights in Xamarin.Forms
date: 2017-02-01 12:23
comments: true
author: adam
tags: [Xamarin.Forms]
---
### Xamarin.Forms Width and Height is -1!?
Occasionally in a Xamarin.Forms project you need access to an object's Width and/or Height and you are being returned a value of -1.  This can be super frustrating!  It however makes perfect sense, because in all of these scenarios the view hasn't gone through a layout cycle preventing it from knowing the size of the object you wish to put on the screen relative to the parent container, etc.

This happens to me a lot when working with an AbsoluteLayout where explicit X/Y positions and Width/Height values are required for positioning and sizing of an object.  For example, I want to center a BoxView inside of an AbsoluteLayout.  Simply putting HorizontalOptions.Center and VerticalOptions.Center does not center the BoxView inside the AbsoluteLayout.

```csharp
_boxView = new BoxView {
	HorizontalOptions = LayoutOptions.Center,
	VerticalOptions = LayoutOptions.Center,
	BackgroundColor = Color.Red,
};
absoluteLayout.Children.Add ( _boxView );
```

The above code results in the following:
<div style="margin: 0px auto;">
<table style="margin: 0px auto;">
	<tr>
		<td align="center">iOS</td>
		<td style="width:100px">&nbsp;</td>
		<td align="center">Android</td>
	</tr>
	<tr>
		<td align="center"><img src="{{site.baseurl}}/images/2017-02-01/iOS-boxview.jpg" /></td>
		<td style="width:100px">&nbsp;</td>
		<td align="center"><img src="{{site.baseurl}}/images/2017-02-01/droid-boxview.jpg" /></td>
	</tr>
</table>
</div>

But why isn't the red box centered!?  This may make sense to someone more experienced using AbsoluteLayout in Xamarin.Forms, but to someone starting out there is no point in using HorizontalOptions or VerticalOptions on an object whose parent is an AbsoluteLayout.  All child objects of AbsoluteLayout must be explicitly assigned their positioning on screen.  You would also find out that if you were to try and Debug.WriteLine the _boxView Width and or Height it is indeed -1.    So how do we get the size of the _boxView object, we wait!

#### Setup a SizeChanged Listener
```csharp
_boxView.SizeChanged += HandleBoxViewSizeChanged;
```

#### Implement the Listener  
```csharp
private void HandleBoxViewSizeChanged( object sender, EventArgs e ) {

	_boxView.SizeChanged -= HandleBoxViewSizeChanged;

	//DO THINGS HERE with _boxView
}
```
Now after the "//DO THINGS HERE.." you have access to the _boxView object's Width and Height (turns our the default BoxView Width/Height is 40)!  Simple right?

Let's go one step further and finish center aligning the BoxView within the AbsoluteLayout as we had hoped the HorizontalOptions and VerticalOptions would have.  I'm also going to give the BoxView a larger size to start, 300x300.

```csharp
private void AddBoxView() {

	_boxView = new BoxView {
		BackgroundColor = Color.Red,
		WidthRequest = 300,
		HeightRequest = 300,
	};
	_boxView.SizeChanged += HandleBoxViewSizeChanged;
	absoluteLayout.Children.Add ( _boxView );
}

private void HandleBoxViewSizeChanged( object sender, EventArgs e ) {

	//ALWAYS UNSUBSCRIBE!
	_boxView.SizeChanged -= HandleBoxViewSizeChanged;

	_boxView.TranslationX = ( absoluteLayout.X + absoluteLayout.Width ) / 2 - ( _boxView.Width / 2 );
	_boxView.TranslationY = ( absoluteLayout.Y + absoluteLayout.Height ) / 2 - ( _boxView.Height / 2 );
}
```

The above code results in the following:
<div style="margin: 0px auto;">
<table style="margin: 0px auto;">
	<tr>
		<td align="center">iOS</td>
		<td style="width:100px">&nbsp;</td>
		<td align="center">Android</td>
	</tr>
	<tr>
		<td align="center"><img src="{{site.baseurl}}/images/2017-02-01/iOS-boxview2.jpg" /></td>
		<td style="width:100px">&nbsp;</td>
		<td align="center"><img src="{{site.baseurl}}/images/2017-02-01/droid-boxview2.jpg" /></td>
	</tr>
</table>
</div>

Wiring up the SizeChanged event handler allowed us to capture the moment the size of the BoxView is updated, and from there we are able to reposition the BoxView to the center doing some basic math.

### Get iOS & Android Image Width & Height Without Loading the Image
Another common need is to have access to the width and height of a Xamarin.Forms Image control prior to loading the image and displaying it on screen.  This needs to be handled in the platform specific code on iOS and Android.  First we'll need an Interface that we can call into via the DependencyService.

#### The Interface
```csharp
public interface IImageInfo {

	Tuple<int, int> GetFileWidthAndHeight ( string file );
}
```

We will call this method from our shared Xamarin.Forms code and pass in the path to the image on disk and it will return a Tuple<int,int> which is our image's width and height.

#### The iOS Implementation
```csharp
using System;
using WidthHeightExamples.iOS;
using ImageIO;
using Foundation;

[assembly: Xamarin.Forms.Dependency ( typeof ( ImageInfo ) )]
namespace WidthHeightExamples.iOS {

	public class ImageInfo : IImageInfo {

		public Tuple<int, int> GetFileWidthAndHeight ( string file ) {

			int width;
			int height;

			using (var src = CGImageSource.FromUrl ( NSUrl.FromFilename ( file ) )) {

				CGImageOptions options = new CGImageOptions ( ) { ShouldCache = false };

				width = ( int )src.GetProperties ( 0, options ).PixelWidth;
				height = ( int )src.GetProperties ( 0, options ).PixelHeight;
			}

			return new Tuple<int, int> ( width, height );
		}
	}
}
```

#### The Android Implementation
```csharp
using System;
using WidthHeightExamples.Droid;
using Android.Graphics;

[assembly: Xamarin.Forms.Dependency ( typeof ( ImageInfo ) )]
namespace WidthHeightExamples.Droid {

	public class ImageInfo : IImageInfo {

		public Tuple<int, int> GetFileWidthAndHeight ( string file ) {

			BitmapFactory.Options options = new BitmapFactory.Options ( );
			options.InJustDecodeBounds = true;

			BitmapFactory.DecodeFile ( file, options );

			int width = options.OutWidth;
			int height = options.OutHeight;

			return new Tuple<int, int> ( width, height );
		}
	}
}
```

In my case I have an PNG image I pulled down from the web and stored in my application's document directory.  This does need to be an image you can get the full path to.  This means that on Android you **cannot** test this with an image you store in your drawable folders as you cannot get the full path to a resource in your drawable directory.

Here is the code:
```csharp
private void AddImage() {
	string filePath = App.AppInfo.DocumentPath + "/mc.png";

	Tuple<int, int> imageWidthAndHeight = DependencyService.Get<IImageInfo> ( ).GetFileWidthAndHeight ( filePath );

	var scale = App.AppInfo.Scale;

	int imageX = ( int )Math.Floor ( ( _boxView.TranslationX + _boxView.Width / 2 ) - ( ( imageWidthAndHeight.Item1 / scale) / 2 ) );
	int imageY = ( int )Math.Floor ( ( _boxView.TranslationY + _boxView.Height / 2 ) - ( ( imageWidthAndHeight.Item2 / scale ) / 2 ) );

	_image = new Image {
		Source = filePath,
		TranslationX = imageX,
		TranslationY = imageY,
		WidthRequest = imageWidthAndHeight.Item1 / scale,
		HeightRequest = imageWidthAndHeight.Item2 / scale
	};

	absoluteLayout.Children.Add ( _image );
}
```

The above code results in the following:
<div style="margin: 0px auto;">
<table style="margin: 0px auto;">
	<tr>
		<td align="center">iOS</td>
		<td style="width:100px">&nbsp;</td>
		<td align="center">Android</td>
	</tr>
	<tr>
		<td align="center"><img src="{{site.baseurl}}/images/2017-02-01/iOS-boxview3.jpg" /></td>
		<td style="width:100px">&nbsp;</td>
		<td align="center"><img src="{{site.baseurl}}/images/2017-02-01/droid-boxview3.jpg" /></td>
	</tr>
</table>
</div>

1. First I get the full path to my image file
2. I call to platform code via my DependencyService (notice the returned Tuple<int, int> is our image on disk's width and height)
3. I grab the scale of the device I am on (this is more common platform code I use a lot so I know the scale factor of the device, for example the iPhone 6 scale is 2, the iPhone 7 Plus is 3)
4. We can then calculate X and Y positions for our image (in this example I'm simply going to center it within the already displayed _boxView)
5. Create the Image control and add it to the AbsoluteLayout

The real magic happened in #2 above where we call into our platform code.  Once we have the width and height of the image on disk you can really start to see the power of having those values available to you as you define your layouts.

Hopefully you learned something new when it comes to accessing an object's Width and Height in Xamarin.Forms.  Obviously the above isn't always needed either.  We don't always need to know the width or height of something and or Xamarin.Forms can simply handle the layouts and adapt them based on the screen size, orientation, etc.  But for some of those really specific needs this can be a great start.

Happy coding!
