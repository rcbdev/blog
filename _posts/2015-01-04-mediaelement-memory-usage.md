---
title:  "Efficient Memory Usage with MediaElement"
date:   2015-01-04
categories: WinRT
description: Efficient memory usage with MediaElement on Windows 8. Why your app is using more memory than it should be and how to fix it.
---

Recently we came across an issue with an app we work on whereby it was crashing on a low memory device (2GB of RAM). This turned out to be caused by the way in which the MediaElement control was being used. If used incorrectly, the presence of multiple MediaElement controls on a single page can cause much higher memory consumption than required.

## The Cause

When the source of a MediaElement is set, it must load the video in memory and prepare to play the video in a smooth way (i.e. support smooth scrubbing through the video). This involves not only loading the video, but also loading many components that help support the playing of a video. This can mean setting the source of a MediaElement will increase the memory usage by more than the size of the video itself. Include multiple videos on a single page in your app, and this can lead to a huge amount of memory being consumed. In our case, the loading of a 250MB file in our app caused 1.3 GB of RAM to be used.

## The Fix

So, if you're experiencing this issue you'll be pleased to know there is a simple fix. Wait to set the source until you need to. If you delay setting the source of the MediaElement, nothing is loaded into memory until it is needed. Of course, if you don't set the source of the MediaElement straight away, you'll be missing features such as the video thumbnail. Luckily, it's easy enough to replace this with your own implementation. By including a placeholder image and a simple play button, you can provide a nice user experience to the end user without requiring a large memory usage.

When using the MediaElement control, especially if you are using multiple at once, I would recommend creating a custom control that does the following:

* Contains a MediaElement, a placeholder image, and a play button
* Loads a thumbnail for the video using `StorageFile.GetThumbnailAsync`
* Sets the source of the MediaElement when the play button is pressed
* Sets the source of the MediaElement to `null` when a "stop" command is issued
* Issue a "stop" command to all other videos on the page when play is pressed

By implementing these features, the memory usage for the app when viewing the 250MB file was reduced from 1.3GB to 200MB. For a very simple change, this is a huge improvement in memory usage.
