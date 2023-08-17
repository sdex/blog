---
layout: post
title: Play a media segment in a loop with ExoPlayer
categories: [Android,ExoPlayer]
---

ExoPlayer provides an easy way to play a selected part of a media that also works for live streams. 
We consider two ways to achieve it.

I use `ExoPlayer` from [Media3](https://developer.android.com/jetpack/androidx/releases/media3), version `"1.1.1"`:

```kotlin
implementation("androidx.media3:media3-exoplayer:1.1.1")
```

In order to configure clipping for the media a `MediaItem` has the `clippingConfiguration` field:

```kotlin
val mediaItem = MediaItem.Builder()
	.setUri(VIDEO_URI)
	.setClippingConfiguration(
		ClippingConfiguration.Builder()
			.setStartPositionMs(0)
			.setEndPositionMs(3000)
			.build()
	)
	.build()
```

To set the media segment we need two methods from `ClippingConfiguration.Builder`: 

`fun setStartPositionMs(startPositionMs: Long)` - sets the start position in milliseconds which must be a value larger than or equal to zero.

`fun setEndPositionMs(endPositionMs: Long)` - sets the end position in milliseconds which must be a value larger than or equal to zero. 
There is a special constant to indicate that the end position is the end of the media -  `C.TIME_END_OF_SOURCE`. This constant is equal to `Long.MIN_VALUE`.

To play the media in a loop we need to set the `repeatMode`:

`exoPlayer.repeatMode = Player.REPEAT_MODE_ONE`

After these steps ExoPlayer starts to play the selected media segment infinitely. 

Under the hood, the player uses `ClippingMediaSource` to clip the media segment, we can use it directly:

```kotlin
val mediaSourceFactory = DefaultMediaSourceFactory(context)
val mediaItem = MediaItem.fromUri(VIDEO_URI)
val mediaSource = ClippingMediaSource(
	mediaSourceFactory.createMediaSource(mediaItem),  
	0,
	3000000
)
exoPlayer.setMediaSource(mediaSource)
```

Note, there is one important difference, `MediaItem.clippingConfiguration` methods take the time in **milliseconds**, whereas `ClippingMediaSource` constructor takes the time in **microseconds**. 1 millisecond is 1000 microseconds. 
To convert you can use the utility method from the media3 library:
`androidx.media3.common.util.Util.msToUs(timeMs: Long)`
