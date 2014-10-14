---
title: 'Sony Smartwatch  SDK Impressions'
author: error454
layout: post
permalink: /2012/06/15/sony-smartwatch-sdk-impressions/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:1:{s:59:"https://mobilecoder.files.wordpress.com/2012/06/capture.jpg";a:6:{s:8:"file_url";s:59:"https://mobilecoder.files.wordpress.com/2012/06/capture.jpg";s:5:"width";s:3:"605";s:6:"height";s:3:"203";s:4:"type";s:5:"image";s:4:"area";s:6:"122815";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"1";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2012-06-15 08:25:24";}'
categories:
  - Android
tags:
  - android
  - extensions
  - sdk
  - smart
  - smartextensions
  - smartwatch
  - sony
  - watch
---
I recently received a Sony Smartwatch as part of <a href="http://developer.sonymobile.com/wp/smartwatch-developer-campaign/" target="_blank">Sony's promotion</a>.  I haven't been this excited about a watch since I was rocking my Casio Calculator Watch back in 2nd grade.

Unlike many SDKs, my first impression when peaking inside the Smart Extensions SDK was that Sony actually has android developers employed!  This excited me since many SDKs feel like Android is a 2nd thought entirely.  Overall the SDK is fairly straight-forward in that reading accelerometer data, getting screen tap coordinates and drawing to the screen are all simple tasks with well commented sample projects.

There were a couple features that I thought were missing and that felt foreign:

*   Having to write button collision detection from scratch rather than using button click handlers
*   Being unable to use XML selectors
*   Being unable to use other Android UI widgets like ProgressDialog

<img src='' alt=''>



## Support

This may seem a small thing, but I like how Sony is approaching developer support for the watch.  I feel that their choice to leverage Stack Overflow was a great one.  I got a response quickly and am also able to do simple searches to see other smartwatch related questions.

## Growing Pains

When I started writing layouts I was momentarily baffled, leading to <a href="http://stackoverflow.com/questions/10974245/xml-layout-on-sony-smartwatch" target="_blank">this SO question</a>.  This turned out to be because my drawables weren't in the drawable-nodpi folder, oops!

## New Implementation for onClickHandler

I was immediately surprised that there wasn't a simple way to load a view on the watch and then have it respond to onClickHandler events like any other android app.  I know that the watch is simply a screen hooked up via bluetooth but I expected to be met with Android-like wrappers for some of the common tasks.

My initial scan through the methods provided by the SDK gave me the impression that everyone was supposed to write their own collision handlers for a simple screen tap.  I was curious how the 8 Game sample did this and no big surprise, that's exactly what they did.  So for instance they have rectangles defined:

<pre>private static final Rect sActionButton1Rect = new Rect(0, 88, 40, 128);
private static final Rect sActionButton2Rect = new Rect(44, 88, 84, 128);
</pre>

Then for each tap event on the watch, they do collision detection:

<pre>if (sActionButton1Rect.contains(event.getX(), event.getY())) {
...
} else if (sActionButton2Rect.contains(event.getX(), event.getY())) {
...
}
</pre>

To me this felt like a huge step backwards in terms of the level of effort required to make even a simple smartwatch app.  Collision handlers aren't difficult to write, but writing one that doesn't break every time you get a bigger screen takes more effort than a simple onClickHandler.  I enjoy the luxury of leveraging higher-level constructs whenever possible, especially if they make my code more resilient to the array of hardware out there.  Sony may announce a 256*256 version tomorrow and now everyone has to rewrite their collision handlers!

With this in mind, I set out to write a layout implementation that would provide the following features:

*   Be able to trigger onClickHandlers attached to a view
*   Be able to use XML selectors to change a drawable when pressed/not pressed
*   Draw logic that could be gated at a specific frame rate that re-used bitmaps to avoid GC

After an afternoon of thought, I decided to implement the following infrastructure:

<img class="size-full wp-image-1128 aligncenter" title="Capture" src="{{ site.url }}/assets/uploads/2012/06/capture.jpg" alt="" width="605" height="203" />

The difference between my approach and the approach of the sample source is this:

1.  The sample source creates an XML layout, renders that layout to a bitmap and then throws that layout away (along with the bitmap) for every screen draw.  This implementation holds onto the layout and bitmap for re-use.
2.  The sample source provides tap events as x, y coordinates.  This implementation forwards tap events to the layout where they are interpreted as android Tap/Hold/Release events that click UI elements as expected.

Initialization for this implementation looks something like this (inside SampleSensorControl):

<pre>mSmartView = new SmartWatchLinearLayout(mContext, WIDTH, HEIGHT);
mLayout = (LinearLayout)LinearLayout.inflate(context, R.layout.pauseplay, mSmartView);
mSmartView.setParameters(mThis, mLayout);

//You can then hookup buttons like usual, the collision detection is handled by the view
mPausePlay = (Button)mSmartView.findViewById(R.id.imageViewPausePlay);
mPausePlay.setOnClickListener(new OnClickListener() {

	@Override
	public void onClick(View v) {
		mXbmc.playPause();
	}
});

//Tell the smartview to monitor this item for touch events
mSmartView.addViewToWatch(mPausePlay);

//You can also request the screen to redraw, the smartview only redraws in response to a touch event
mSmartView.requestDraw();
</pre>

Finally, plumb-in the onTouch events and you're ready to rock:

<pre>@Override
public void onTouch(ControlTouchEvent event) {
	super.onTouch(event);

	//Forward this touch event on to the smartview
	mSmartView.dispatchControlTouchEvent(event);
}
</pre>

Dispatching tap events to the layout allows Android to trigger selectors and onClickHandlers.  Essentially you get to circumvent all of the collision detection!  This isn't perfect, but it is a pattern I'll be using on my next smartwatch project, if nothing else, to use click handlers.

I'm still not fully satisfied with the draw loop of the implementation.  I'd like to completely eliminate the need to manually request drawing but so far haven't found a solution that does so reliably without being a resource hog.

If you'd like to see the full implementation, you can grab the source for my <a href="https://github.com/error454/xbmc-smartwatch" target="_blank">XBMC Smart Extension</a>.

## Final Thoughts

I like the app possibilities that the Smartwatch brings to the table.  The SDK might feel a little bare-metal to casual Android developers but there are enough samples to get by on.  I would most like to see some kind of wrappers for the most basic Android UI Widgets like ProgressDialog.