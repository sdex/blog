---
layout: post
title: Add a video fade effect on Android using FFmpeg and Media3 transformer (OpenGL texture)
categories:
  - Android
  - Media
---
In this article, we look at two ways of adding video fade-in and fade-out transition effects. 
### FFmpeg

Adding a fade effect is quite simple with FFmpeg fade filter:
https://ffmpeg.org/ffmpeg-filters.html#fade

We need to set 3 parameters: type (in/out), start time, and duration.

Add fade-in to the beginning of the video:
`fade=t=in:st='0ms':d='2000ms'`

Add fade-out to the end of the video (assuming the video is 7 seconds long):
`fade=t=out:st='5000ms':d='2000ms'`

It works like a charm but there are some drawbacks to consider. 
FFmpeg re-encodes the video and often the output file is quite different from the original, software encoders are much slower than hardware, increased app size, etc. 

### OpenGL texture

The idea is simple - draw a black image on the top of the each frame and adjust the image transparency depending on the frame time. 
To achieve it using the `MediaCodec` we need to:

1. Extract frames from the video using `MediaExtractor`. 
2. Pass the frames to `MediaCodec` decoder.
3. Create an output surface.
4. Render the video frames. 
5. Render the overlay image. 
6. Encode the frames using `MediaCodec` encoder. 
7. Save the result video using `MediaMuxer`.

An easier way is to use the Media3 Effect library which hides all the complexity and offers a simple way to add video effects. 
I created the `FadeOverlay` class that extends `BitmapOverlay`, it creates a single bitmap with the same size that video, and paints it black. Then returns the image in the overridden method:

```kotlin
override fun getBitmap(presentationTimeUs: Long): Bitmap = overlayBitmap
```

In the `getOverlaySettings` method it uses `presentationTimeUs` to calculate `alphaScale` and return the `OverlaySettings` for the given time: 

```kotlin
override fun getOverlaySettings(presentationTimeUs: Long): OverlaySettings {  
    if (presentationTimeUs < fadeInDuration) { // fade in  
        val alphaScale = ...  
        return OverlaySettings.Builder()  
            .setAlphaScale(alphaScale)  
            .build()  
    } else if (presentationTimeUs > videoDuration - fadeOutDuration) { // fade out  
        val alphaScale = ...  
        return OverlaySettings.Builder()  
            .setAlphaScale(alphaScale)  
            .build()  
    }  
    return defaultOverlaySettings  
}
```

Finally, use `Transformer` to apply the effect and process the video:

```kotlin
val fadeOverlay = FadeOverlay(...)
val videoMediaItem = MediaItem.fromUri(uri)
val videoEffects = listOf(
    OverlayEffect(ImmutableList.of(fadeOverlay))
)
val audioProcessors = emptyList<AudioProcessor>()
val videoEditedMediaItem = EditedMediaItem.Builder(videoMediaItem)
    .setEffects(Effects(audioProcessors, videoEffects))
    .build()
val transformer = Transformer.Builder(context)
transformer.start(videoEditedMediaItem, outputFile.absolutePath)
```

The result video: 

{{< youtube "x37_Xsrhlw0" >}}
