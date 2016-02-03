---
layout: post
title: Remote debugging with Chrome and Android
category: blog
---

Chrome's DevTools tray has a heap of useful features, including device emulation, but sometimes a physical device is a heap more useful.

For that, Chrome has a nifty feature which provides a mechanism to remotely debug an Android device on your desktop or laptop development machine.

There's a [quick setup guide on the Google Developers site](https://developers.google.com/web/tools/chrome-devtools/debug/remote-debugging/remote-debugging), but despite being really good at following instructions, I couldn't get it working.

If you're not so good at following instructions, the crux of the Google walkthrough is as follows:

- Fire up your Android device
- Head to Settings -> Developer options (this may not be visible - if not, head over to About device -> find the build number -> tap it 7 times. Yup. Seriously)
- Back in Developer options, enable USB debugging
- Fire up Chrome on the dev machine, head to chrome://inspect
- Make sure the 'Discover USB devices' checkbox is ticked
- Your Android device should be listed, with a note about pending authentication
- Back on the device, you'll see a popup asking for permission to allow the remote connection
- Click OK
- Open Chrome on the device, and debug your sweet little heart out

Unless you're like me, and it simply doesn't work.

In that case, try this:

- Install the [Universal ADB driver](https://github.com/koush/UniversalAdbDriver)
- Download the [Android SDK platform tools](https://www.androidfilehost.com/?fid=9390355257214632011)
- Extract the archive, right click in the folder, run command window here
- In the console, run this command: abd devices
- If you're still like me, you'll see a message about the daemon not running and then starting successfully.
- Close the command window

From there, repeat the initial steps, and all should be sweet. Except when you realise the issue you'd wanted to debug appears to have been fixed in an unrelated task and is no longer an issue and you've spent half a day swearing at a stack of mobile devices for nothing. 

Note: The ADB driver may not be required - I'd installed it before grabbing the platform tools, but hey, up-to-date drivers can't hurt.

 
