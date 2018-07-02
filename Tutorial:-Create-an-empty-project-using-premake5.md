This guide aims to help you easily create an empty SFML project using premake5.

## Prerequisites:
* [premake5 portable executable](https://premake.github.io/)
* SFML already built (or [prebuilt](https://www.sfml-dev.org/download.php)) statically
* [SFML's premake script](https://gist.github.com/gale93/b37fe2b3194a150dca09d122f9ef6c3d)

## How to

You have to setup your directories like this:

![](https://image.ibb.co/euOAWd/screen.png)

Put under **dependencies** your SFML folder ( without the version ). It will lookup for **SFML/include** and **SFML/lib** directories only.

Put a symbolic _main.cpp_ under **src** directory ( Use [this](https://www.sfml-dev.org/documentation/2.5.0/) or whatever you like )

Now just:

`premake5 vs2017` ( if you are using visual studio 2017, otherwise the complete list is [here](https://github.com/premake/premake-core/wiki/Using-Premake))

And under the new generated folder **build** you will find your project ready to go.

## FAQ
* ### Getting strange link errors on visual studio 2017

It happens that it will take the wrong version of the Windows' SDK, simply select the latest under Project's Properties ->
**Configuration Properties/General**
