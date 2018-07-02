Porting to Android - MonoGame, Textures and SpriteSheets
========================================================

Submitted by Beringela

on Sun, 03/31/2013 - 17:42

Introduction
------------

When you're porting your XNA game to Android using MonoGame, if your game is of any size and complexity, you may well come across the issue of textures showing up all black (or all white, or all transparent) after a tombstone/restore. In fact, if all your textures live in SpriteSheets then you may find the entire screen goes black after a restore. If you've got some textures you create with Texture2D.SetData or otherwise generate on-the-fly without using a ContentManager, then you may also find those are black after a tombstone/restore. Note Android doesn't officially refer to tombstoning as tombstoning, but I will still use that because it is terminology we're all used to from Windows Phone. This blog explains how to fix these black textures.

Problem Statement
-----------------

If you're using the latest version of MonoGame, you should find that any Texture2D or SpriteFont assets that are managed by a ContentManager will be handled just fine after a tombstone/restore. They retain their texture and don't go black. If however, you use a custom type like a SpriteSheet with a ContentManager, or you create Texture2Ds yourself programatically, then you may find these are black after a restore. The root cause is not MonoGame (it is something deeper, either Android itself or Mono) but MonoGame does provide us the tools to fix it. The images below show the game running before and after a tombstone/restore, and you can see the black texture problem clearly :):

![](https://web.archive.org/web/20170513233133im_/http://www.pencelgames.com/sites/default/files/styles/large/public/201303/BlackTextures_GerbilBefore.png?itok=gSH48T0E)

![](https://web.archive.org/web/20170513233133im_/http://www.pencelgames.com/sites/default/files/styles/large/public/201303/BlackTextures_GerbilAfter.png?itok=IkObjrKu)

Solution
--------

Thanks to the helpful folks [on the MonoGame forums](https://web.archive.org/web/20170513233133/http://monogame.codeplex.com/discussions) (special thanks to SlyGamer, Aranda and Murudai) these issues are all resolvable. To save you trawling through many forum posts to piece this all together, I'm going to summarise their wisdom and demonstrate the solution all here in one place.

Part 1 - Porting XNA SpriteSheets to MonoGame for Android
---------------------------------------------------------

Most of the points I want to cover are all demonstrated in this zip file, which is a port of the [XNA SpriteSheet sample](https://web.archive.org/web/20170513233133/http://xbox.create.msdn.com/en-US/education/catalog/sample/sprite_sheet) to MonoGame for Android. This sample still works for Windows Phone but the main projects have been ported to Android so you can see what changes are needed:

**[XNA SpriteSheet sample ported to MonoGame for Android](/web/20170513233133/http://www.pencelgames.com/sites/pencelgames.drupalgardens.com/files/201303/SpriteSheetSampleAndroid.zip)**

### Whats in the Zip file?

In root of the zip file you've got a "SpriteSheetSample (Phone).sln". This contains the content pipeline and the runtime example for Windows Phone. You can build and run this as normal from Visual Studio and see the sample working on the Windows Phone emulator.

In general, I've added an "A" on the end of projects or solutions that are built for Android. In the solution "\\SpriteSheetSampleA\\SpriteSheetSampleA.sln" you've got the Android runtime example. This relies on the XNB being built for it (already done and included the zip file) by the phone project above. To open the Android solution you'll need MonoDevelop or Xamarin Studio. The sample will run on the Android emulator or an Android device.

### Interesting things to note:

1) Note the Assembly name of the SpriteSheetRuntimeA project is still SpriteSheetRuntime. This is necessary so the correct SpriteSheet Reader can be found.

2) You can see that this sample has our own custom SpriteSheetWriter and SpriteSheetReader. (If you need a refresher on the content pipeline [Shawn Hargreaves has some good articles](https://web.archive.org/web/20170513233133/http://blogs.msdn.com/b/shawnhar/archive/2008/11/24/content-pipeline-assemblies.aspx). The custom content writer (class SpriteSheetWriter) is called at Content Build time and embeds the name of our custom content reader (class SpriteSheetReader) in the XNB. The custom Reader has code which handles the reloading of texture assets whilst preserving other data. See the line:

if (existingInstance != null)

in the SpriteSheetReader class. That is where all the work is done to restore textures after a tombstone/restore.

3) The MonoGame folks have extended XNA so that a new method called ContentManager.ReloadGraphicsAssets() is called when an Android device is tombstoned/restored. The MonoGame code in ContentManager.ReloadGraphicsAssets() recognises SpriteFonts, Texture2Ds, but it obviously doesn't know anything about custom content types, like our SpriteSheet! To get around this, we can subclass the ContentManager, and explicitly get it to reload the graphics assets of our SpriteSheet. You can see this in the ContentManagerSpriteSheet class in the SpriteSheetRuntimeA project. (An alternative could be to make our SpriteSheet a subclass of Texture2D, but I didn't try that route.)

This "SpriteSheetSampleA.sln" sample correctly reloads the SpriteSheet after a tombstone/restore. If you want to see the black-texture-after-restore bug happening, simply comment out the line that says:

ReloadAsset<SpriteSheet>(asset.Key, asset.Value as SpriteSheet);

in the ContentManagerSpriteSheet class. With that line commented out, the textures won't get reload when the sample is resumed. So run the sample, press the home button on your Android device, and navigate to the sprite sheet sample icon again and tap it to resume the sample. The spritesheet textures will be black.

Part 2 - Porting XNA Textures not created by a ContentManager
-------------------------------------------------------------

You may well have opted to create some textures directly in code. In our game we did some building-a-Texture2D-in-code to shade some letter backgrounds like this:

![](https://web.archive.org/web/20170513233133im_/http://www.pencelgames.com/sites/default/files/styles/large/public/201303/BlackTextures_LetterBefore.png?itok=Yr7wkDzC)

This allowed us to create nice gradations without knowing the width of the letter beforehand. After a tombstone/restore, though, the letter looked like this:

![](https://web.archive.org/web/20170513233133im_/http://www.pencelgames.com/sites/default/files/styles/large/public/201303/BlackTextures_LetterAfter.png?itok=bX-PU1kh)

Or occasionally transparent. The letter itself was preserved perfectly because it is a SpriteFont, and the sky behind was preserved perfectly because it is a Texture2D loaded from a ContentManager.

So how do we fix the letter shading textures? We have to listen to the GraphicsDevice.DeviceReset event, and on hearing it, we have to rebuild the texture. The GraphicsDevice is visible from your Game object, or any DrawableGameComponent. I won't show the code for that, because it's going to be very specific to your own solution, but this technique does fix the issue.
