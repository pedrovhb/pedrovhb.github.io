---
title: "Sometimes you gotta fix it yourself"
date: 2021-11-18T16:44:35-03:00
draft: false
---

I'm a huge motorsports fan, and this weekend I had the privilege of attending the 2021 São Paulo Grand Prix. It was an amazing Formula 1 race in the highly regarded Interlagos circuit, and it'll go down as one of the classic races of the sport, with a performance from Lewis Hamilton that was nothing short of breathtaking.

COVID is still a concern in sports events such as this, and reasonably enough, F1 management elected to use a digital solution to verify vaccination status for anyone who attended. I've had my two shots, so I was initially at ease. However, when trying to log in to the app used by the organization, I was quite concerned by the fact that I couldn't complete my registration! 

Now, it seems several people had issues with the app, and I'm sure they loosened the enforcing and accepted plain vaccination cards, but I wasn't about to take a chance on having a big headache in a weekend that should be a high point of my year. I of course did what any sane (or perhaps just curious) developer would do, and set out to do some investigating and try to fix the problem. 

Unfortunately I didn't take screenshots then, but what happened was that when trying to submit a photo, which was necessary to finish registration, nothing happened. I tapped the button to open the camera, multiple times, but there was no indication in the UI that anything was going on; it seemed to just ignore me. I haven't done much Android development at all, but I've played around with its innards a bit in the past, and I had the idea to check logcat to see if that spit out something more useful. I used [LogCat Extreme](https://forum.xda-developers.com/t/app-2-1-logcat-extreme.1513166/) to create a floating log window, and sure enough, when I tapped the "Open camera" button, an error message was logged -

> ```
> E/flutter: [ERROR:flutter/lib/ui/ui_dart_state.cc(177)] Unhandled Exception: 
> PlatformException(no_available_camera, No cameras available for taking pictures)
> ```

That's weird, I'm pretty sure my phone has more cameras than I have fingers in my hand. I checked permissions, but they were all there - I even granted some that the app didn't ask for, just in case, but nothing came out of it.

A quick search revealed a [promising lead](https://stackoverflow.com/a/64693774):

> If your compileSdkVersion and targetSdkVersion is 30 (or above), then add the queries info to your Android manifest file, directly under the manifest tag:
>
> ```
> <manifest package="your.app.package.name">
>     <queries>
>           <intent>
>                <action android:name="android.media.action.IMAGE_CAPTURE" />
>           </intent>
>     </queries>
> </manifest>
> ```

That's interesting. Digging a bit more into that, I found out that this is something that's required of apps from Android 11 onwards. Seems to make sense: it's possible the developers only tested their app on older versions of Android, and mine is 11, so it checks out that it'd be affected. I used [App Manager](https://muntashirakon.github.io/AppManager/) to view the app's `AndroidManifest.xml`, and indeed that snippet was not present.

Unfortunately, modifying that file isn't as simple as viewing it. After reading up a bit on how the Android apk compilation process worked, I was somewhat confident that it would be possible to do it though, so I set out to try.

I downloaded the app installer file from a mirror website, and it turned out to be an `.xapk` file, which is essentially just a zip archive containing multiple `.apk` files. Using `unzip` and the great `apktool`, I extracted those and decompiled the app files:

```
$ unzip Chronus_v1.0.1_apkpure.com.xapk
$ apktool d com.chronus.ipassport.apk
```

Among the decompiled files was the pesky `AndroidManifest.xml` I had been looking for, and I used a simple text editor to add in the required intent lines mentioned above.

Now we rebuild the app files with `apktool b`, try to install, and...

```
Failure [-124: Failed parse during installPackageLI: 
Targeting R+ (version 30 and above) requires the resources.arsc of installed APKs to be stored uncompressed and aligned on a 4-byte boundary]
```

Alrighty. Luckily I had just read something about this alignment thing, so I knew roughly what to look for. In the end, `zipalign` saved the day when I applied it to all built apk files (pythonic loop made possible by [xonsh](https://xon.sh/)):


```
 for f in `.*.apk`:
     print("Aligning", f)
     $(zipalign -v 4 @(f) @(f + "_aligned.apk"))     
```

I then had to sign the apk files so my phone would install them, which I did with a debug key I generated on the spot -

```
apksigner sign --ks ../debug.keystore config.xhdpi.apk_aligned.apk
apksigner sign --ks ../debug.keystore config.arm64_v8a.apk_al.apk
```

And finally, since this was an apk bundle, I had to use `adb install-multiple` to install the apk files which I eyeballed to be relevant to me:
```
adb install-multiple com.chronus.ipassport.apk_aligned.apk  config.pt.apk_al.ignedapk config.xhdpi.apk_aligned.apk config.arm64_v8a.apk_aligned.apk
```

I booted up the app, fingers crossed, and... Success! I was able to finish the registration, and I obviously had to take the required picture with the smuggiest smirk I could muster :)

![Screenshot of the app after logging in](/img/2021/fix_it_yourself/cornus.jpeg)

I'm happy to report that all went perfectly smooth at the venue, and I had no problem having my vaccine log validated. I got to enjoy an amazing race, the nice feeling of having solved a problem with my 1337 skillz, and having a cool, if a bit nerdy, story to share!

I passed on the tip of how to fix the issue to the official Grand Prix page, but they haven't responded to that either. As of the time of writing, the app [hasn't been updated](https://play.google.com/store/apps/details?id=com.chronus.ipassport&hl=en&gl=US) since March and still has a 1.2-star rating, which is actually impressive, considering the minimum rating is 1 star.

Hopefully we can have more reliable software in the future. Until then, we may have to fix things ourselves every once in a while. The plus side is that at least we get to practice our smug smiles while doing it :)  



