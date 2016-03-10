I previously found the lack of a solid tutorial difficult, ending up spending around 5-6 days attempting to get SFML.NET to work on OSX. I finally got it to work after trudging through problem after problem. So I decided to write a tutorial to help newer users get setup on OSX.

I'm going to assume you'll be using Xamarin IDE. If you don't, it's a great tool for working with C# on OSX, and I highly recommend you use it.

# Resources

If you're using the latest SFML.Net version (2.2), you'll want to download the following libraries for your project.

* SFML.Net - http://www.sfml-dev.org/download/sfml.net/ (2.2 64bit)
* CSFML - http://www.sfml-dev.org/download/csfml/ (2.2 MacOSX Clang Universal)
* SFML - http://sfml-dev.org/download.php (2.2 MacOSX Clang Universal) 
* InstallNameTool - https://code.google.com/p/install-name-tool-gui/ (Prebuilt Binary) with Command Line Tools installed.

# Project Setup

Once you've created a new Command Line project from Xamarin, you can go ahead and add the SFML.Net bindings by going into your references and adding the contents of the _lib_ folder inside the sfmlnet zip file. This should be three DLL's.

* sfmlnet-audio-2.dll
* sfmlnet-window-2.dll
* sfmlnet-graphics-2.dll

Just select them in the lib directory, and click add.

![Add references](http://jamiehoyle.com/tfg/sfmltut2.png)

# CSFML Setup

The SFML.Net binding does not use the SFML library directly. Instead, it uses the CSFML binding, so we have to add this to our binary directory. Copy the following files from the _lib_ directory inside the CSFML zip you downloaded into your _bin/Debug/_ directory.

* libcsfml-audio.2.2.0.dylib
* libcsfml-window.2.2.0.dylib
* libcsfml-graphics.2.2.0.dylib

![CSFML Setup](http://jamiehoyle.com/tfg/sfmltut3.png)

# SFML Setup

Now that we have added the CSFML binding, all that's left to add is the actual SFML library itself.  Copy the following files from the _lib_ directory inside the SFML zip you downloaded into your _bin/Debug/_ directory.

* libsfml-audio.2.2.0.dylib
* libsfml-graphics.2.2.0.dylib
* libsfml-system.2.2.0.dylib
* libsfml-window.2.2.0.dylib

![CSFML Setup](http://jamiehoyle.com/tfg/sfmltut4.png)

You'll also have to copy the external libraries used by SFML from the _extlibs_ folder into your _bin/Debug/_ directory as well.

* freetype.framework
* sndfile.framework

![CSFML Setup](http://jamiehoyle.com/tfg/sfmltut5.png)

# Basic Application

In order to make sure everything is working fine, replace the contents of _Program.cs_ with the following example code.

```csharp
using System;
using SFML;
using SFML.Graphics;
using SFML.Window;

namespace TestSFML
{
	class MainClass
	{
		private static RenderWindow _window;

		public static void Main(string[] args)
		{
			_window = new RenderWindow(new VideoMode(800, 600), "SFML window");
			_window.SetVisible(true);
			_window.Closed += new EventHandler(OnClosed);
			while (_window.IsOpen)
			{
				_window.DispatchEvents();
				_window.Clear(Color.Red);
				_window.Display();
			}
		}

		private static void OnClosed(object sender, EventArgs e)
		{
			_window.Close();
		}
	}
}
```

If you're doing good so far, the program should build successfully. When you attempt to run it, you should encounter the following error. This is due to the method in which the binding maintains compatibility with all of the supported platforms. If you were running on windows, you would be able to drop in the required dll's, and mono would be able to correctly reference them. Since we're using mono on OSX, we have to actually map the dll's to the correct dynamic libraries. This is very easy to do.

![CSFML Setup](http://jamiehoyle.com/tfg/sfmltut6.png)

Create a file named _sfmlnet-graphics-2.dll.config_ inside your _bin/Debug/_ folder and fill it with the following contents.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <dllmap dll="csfml-graphics-2" target="libcsfml-graphics.2.2.0.dylib"/>
    <dllmap dll="csfml-audio-2" target="libcsfml-audio.2.2.0.dylib"/>
    <dllmap dll="csfml-window-2" target="libcsfml-window.2.2.0.dylib"/>
</configuration>
```

This will redirect the libraries properly, and mono should be able to find them.

# Linking

If you're done the previous steps, you'll notice that mapping the dlls still results in errors. This is due to the way SFML finds the other libraries. Fixing this is possible, but difficult through the command line. Using the InstallToolGUI binary which you downloaded, you can easily fix the relative paths. First though, we need to be able to read the errors that mono produces in-case we do something wrong. This needs to be done via the command line. Start by changing the directory to your project binary folder.

```
> cd ~/Projects/TestSFML/TestSFML/bin/Debug
```

You can now debug your setup by typing the following command, which will output a block of text which should explain why your application may not be able to find the dll. If an error occurs loading or linking the dynamic library, the error will still tell you it cannot find it. Even if it exists.

```
> MONO_LOG_LEVEL=debug mono TestSFML.exe
> ...
```

If you read the log, it's not clear why it cannot find the library. While the filename is correct, the actual id is not. In the next step, we'll go through and fix this.

# Fixing library paths

We'll start by fixing the CSFML binding, start the InstallNameToolGUI application and open _libcsfml-graphics-2.1.dylib_ in your binary directory. Firstly, change the dynamic library ID so that it matches the filename correctly. Press the **Set ID** button to save. You should then remove all the prefix paths in the list, and make sure it is the correct filename. Everything in the CSFML files should reference the SFML libraries.

Here is an example.

![CSFML Setup](http://jamiehoyle.com/tfg/sfmltut8.png)

Once you are done changing all the CSFML dynamic libraries, you can progress by altering the SFML libraries in a similar way. Again, renaming the dynamic library ID and altering all the relative paths. In both graphics and audio, you will have to alter the **freetype** and **sndfile** frameworks paths.

Here is an example.

![CSFML Setup](http://jamiehoyle.com/tfg/sfmltut9.png)

# You're done!

If you've done everything correctly, you will end up with a rather satisfying window on your screen.

![CSFML Setup](http://jamiehoyle.com/tfg/sfmltut10.png)