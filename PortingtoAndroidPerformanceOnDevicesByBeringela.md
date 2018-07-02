Porting to Android - Performance on Devices
===========================================

Submitted by Beringela

on Sun, 03/24/2013 - 15:13

We've started porting Gerbil Physics to Android, which brings us into strange and unfamiliar territory. In the lovely world of Windows Phones, you are guaranteed a minimum spec device and screensize, but not so with Android, which has myriad screen sizes, aspect ratios, CPUs, memory, and many many different operating system versions.

Here are some things we've learnt recently about performance on Android:

*   Performance on the Android Emulator is not representative of how it'll perform on the device. There is an important difference between an Emulator and a Simulator. A Simulator would be a perfect simulation of an Android device, including the precise performance. An Emulator, which is what is provided with the Android SDK, only emulates (is "a bit like") the device, and performance is all down to the device you run it on, i.e. your PC. Fast PC means fast Emulator. So in summary - the Emu tells you pretty much nothing about how your game will perform on an actual device.
*   If you are using Windows, you should [use the Intel x86 device images](http://developer.android.com/tools/devices/emulator.html) when you use the Emulator, otherwise if you use the ARM ones the Emulator will be unplayably slow.
*   We first tried the game on a [Samsung Galaxy Ace](http://en.wikipedia.org/wiki/Samsung_Galaxy_Ace), and it was running at about 0.5 fps or less. Unplayable. The problem is because older Android devices have CPUs running ARM v6 instruction sets (or earlier) and this doesn't support hardware floating point. Any floating point calculations have to be done in software. Games in general, and physics engines in particular, make plenty use of floating point calculations.
*   We tried again on a [Samsung Galaxy Ace 2](http://en.wikipedia.org/wiki/Samsung_Galaxy_Ace_2), and it's CPU ([a NovaThor U8500](http://en.wikipedia.org/wiki/NovaThor)) runs ARM v7. Now we get hardware floating point and the game runs beautifully at a smooth 30 fps. Hurrah!
*   If you're using MonoDevelop, you've got a project option if you look at Project Options -> Build -> Android Build -> Advanced, where you can specify what ABIs you support. If you ONLY tick ARMEABI-V7A then you're game will generate ARM v7 CPU instructions and can use hardware floating point. In addition, your game won't be visible in the Google Play marketplace to phones that cannot run that instruction set.

![](https://web.archive.org/web/20170908040425im_/http://www.pencelgames.com/sites/default/files/201303/ARMEABI_0.png)
