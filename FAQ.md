# Frequently asked questions (FAQ)
**[General](#general)**
- [What is SFML?](#grl-whatis)
- [On which platforms is SFML currently available?](#grl-platforms)
- [Which programming languages are supported by SFML?](#grl-languages)
- [What dependencies does SFML have?](#grl-dependencies)
- [What version of SFML should I use?](#grl-version)
- [Is there a complete list with all the changes from SFML 1.6 to SFML 2.x?](#grl-changes)
- [Will/does SFML support 3D?](#grl-3d)
- [I want to propose a new feature!](#grl-reqeust)
- [Where can I ask questions?](#grl-questions)

**[Building and Using SFML](#build-use)**
- [How do I build SFML?](#build-build)
- [Are there any nightly builds?](#build-nightly)
- [How do I setup my development environment to work with SFML?](#build-environment)
- [I want to fuse the libraries into one. (Not recommended)](#build-fuse)
- [What and how do I link to use SFML?](#build-link)

**[SFML Graphics](#graphics)**
- [What image formats does SFML support?](#graphics-image-formats)
- [Why do I get a white/black rectangle instead of my texture?](#graphics-white-rect)
- [What is the difference between sf::Image and sf::Texture?](#graphics-image-texture)
- [My FPS count drops when drawing a lot of sprites.](#graphics-sprites)
- [Should I use VSync, window.setFramerateLimit or something else?](#graphics-vsync-framelimit)
- [Should I use one sprite or x sprites to draw x textures?](#graphics-xsprite)
- [What is the difference between LocalBounds and GlobalBounds?](#graphics-bounds)
- [My FPS is very low when running my application.](#graphics-low-fps)
- [RenderTextures don't work on my computer!](#graphics-broken-rendertexture)

**[SFML Audio](#audio)**
- [What audio formats does SFML support?](#audio-formats)

**[SFML Networking](#network)**
- [How do I create &lt;insert popular application type here&gt;?](#network-create-network-app)
- [Should I use TCP or UDP sockets?](#network-tcp-vs-udp)
- [Should I use blocking or non-blocking sockets?](#network-blocking-non-blocking)
- [How do selectors work?](#network-selectors)
- [I can't connect to the other computer over the internet!](#network-internet-network)

**[SFML Window](#window)**
- [How do I make my window transparent?](#window-transparent-window)
- [What happened to getFrameTime()?](#window-get-frame-time)

**[SFML System](#system)**
- [Does SFML support Unicode?](#system-unicode)
- [How do I convert from sf::String to &lt;type&gt; and vice-versa?](#system-string-convert)
- [My program keeps crashing when I use threads!](#system-threads-crash)
- [How do I use sf::Mutex?](#system-mutex)
- [Why can't I store my sf::Thread in an STL container?](#system-thread-container)

**[Programming in General](#programming)**
- [What is RAII and why does it rock?](#prog-raii)
- [Why shouldn't I manually manage my memory?](#prog-mmm)
- [What are smart pointers?](#prog-smart)
- [Why shouldn't I use global variables?](#prog-global)
- [What should I use then instead of global variables?](#prog-insteadof-global)
- [Why is the singleton pattern not a good one?](#prog-singleton)

**[Troubleshooting](#trouble)**
- [General](#tr-grl)
 - [I'm having trouble using SFML.](#tr-grl-trouble)
 - [I keep getting "undefined reference to &lt;some strange thing that looks like an SFML function&gt;"!](#tr-grl-undefined-ref)
 - [My computer crashes when I run my SFML program!](#tr-grl-crash)
 - [I found a bug!](#grl-grl-bug)
 - [What is minimal code?](#tr-grl-minimal)
 - [And how can I easily obtain this minimal code?](#tr-grl-obtain-minimal)
- [Code::Blocks](#tr-cb)
 - [I've recompiled the static version of SFML and I'm getting linker errors.](#tr-cb-static-linker)
 - [I'm getting linker errors.](#tr-cb-linker)
- [Visual Studio](#tr-vs)
 - [My project crashes randomly, but I don't get any compiler or linker errors.](#tr-vs-crash)
- [Windows](#tr-win)
 - [Why does a console attach itself to my project?](#tr-win-console)
- [Linux](#tr-lnx)
 - [(Debian) I can't compile the source code.](#tr-lnx-compile)
 - [There is no titlebar visible and/or artifacts from windows are visible.](#tr-lnx-titlebar)

**[Licensing](#licensing)**
- [What license does SFML have?](#lic-license)
- [Can I use SFML in commercial applications?](#lic-commercial)
- [Can I link SFML statically?](#lic-static)
- [Can I use the code from the example directory?](#lic-examples)
- [Do I have to pay any license fees or royalties?](#lic-pay)

**[Libraries for SFML](#libraries)**
- [Does SFML have a GUI package?](#libraries-gui-package)
- [Can you interface SFML with a GUI library?](#libraries-gui-lib)
- [Can I read video files with SFML?](#libraries-video)
- [What exactly is Thor?](#libraries-thor)

**[Miscellaneous](#misc)**
- [Are there any famous projects with SFML?](#misc-projects)
- [How can I distribute my game?](#misc-distr)
- [Where can I upload my game to?](#misc-upload)
- [Are there any example projects I can learn from?](#misc-examples)

---

## <a name="general"/>General

### <a name="grl-whatis"/>What is SFML?

SFML is a simple to use and portable API, written in C++. You can think of it as an object oriented SDL. SFML is made of modules in order to be as useful as possible for everyone. You can use SFML as a minimalist window system in order to use OpenGL, or as a complete multimedia library full of features to build video games or multimedia softwares.

You can find a more specific presentation of its features on [this page](http://www.sfml-dev.org/features.php).

### <a name="grl-platforms"/>On which platforms is SFML currently available?

SFML is currently available and fully functional in Windows (8, 7, Vista, XP, 2000, 98), Linux and Mac OS X. SFML works on both 32 and 64 bit systems.

### <a name="grl-languages"/>Which programming languages are supported by SFML?

SFML is implemented in C++. That said, several bindings have been created for other languages that allow SFML to be used from C, C#, C++/CLI, D, Ruby, OCaml, Java, Python and VB.NET.

### <a name="grl-dependencies"/>What dependencies does SFML have?

SFML depends on a few other libraries, so before starting to compile you must have their development files installed.

On Windows and Mac OS X, all the needed dependencies are provided directly with SFML, so you don't have to download/install anything. Compilation will work out of the box.

On Linux however, nothing is provided and SFML relies on your own installation of the libraries it depends on. Here is a list of what you need to install before compiling SFML:

* pthread
* opengl
* xlib
* xrandr
* freetype
* glew
* jpeg
* sndfile
* openal

The exact name of the packages depend on each distribution. And don't forget to install the development version of these packages.

SFML has also internal dependencies: Audio and Window depend on System, while Graphics depends on System and Window. In order to use the Graphics module, you must link with Graphics, Window, and System (the order of linkage matters with GCC).

### <a name="grl-version"/>What version of SFML should I use?

The short answer: 2.0.

The long answer:
Officially, the released and stable version of SFML is still 1.6, and that is also the version that you are likely to find in many package managers. The reason is that most of them have policies dictating that they do not include software that is still in development because they need to guarantee to their users that it is in the most stable form it can be.
This however is less applicable to SFML than it is to other more volatile projects that are developed rapidly and are indeed very unstable until officially released. SFML 2.0 has been under development for well over a year and has reached RC status. This means it is as good as released concerning stability and it's feature set and API probably won't change at all between now and release.
So if you need to choose, definitely go for 2.0. It will save you a lot of headaches because 1.6 is not actively maintained anymore and if you start with 2.0 you can be sure that the migration to the release build will be as smooth as possible when it comes out.

### <a name="grl-changes"/>Is there a complete list with all the changes from SFML 1.6 to SFML 2.x?

This non-exhaustive list can be used as a starting point: (http://en.sfml-dev.org/forums/index.php?topic=5343.0)
It however does not contain all changes made between 1.6 and 2.0. It was written more than a year ago and since then a few major changes have been made including:

* Rewrite of the graphics API
* New sf::Time API
* Removal of the default built-in Arial font
* Replaced getWidth() and getHeight() with getSize()
* Naming convention change

### <a name="grl-3d"/>Will/does SFML support 3D?

No, and Laurent (SFML's developer) has decided to keep the library as a way to handle 2D graphics with ease and hardware acceleration, so in short there won't be support for 3D in the future either. However you can use Irrlicht with SFML as a window creator. You could also use raw OpenGL to implement 3D and have it alongside your 2D rendering in SFML without problems. 

The previous statement is recommendable only if you have a minimal use for 3D, as it becomes very hard and tedious to manage full 3D functionality through graphics pipeline only. 

### <a name="grl-reqeust"/>I want to propose a new feature!

Before anything else, check the [road-map](https://github.com/LaurentGomila/SFML/issues/milestones) to see if the functionality has already been planned. If not, there is a [forum section](http://en.sfml-dev.org/forums/index.php?board=2.0) dedicated to feature requests. Please search before posting, and stick to the spirit of SFML as a multimedia and multi-platform library. So for example a XML parser, a database library or a platform-specific function is unlikely to be accepted.

### <a name="grl-questions"/>Where can I ask questions?

Post any questions you have regarding SFML in the [SFML forum](http://en.sfml-dev.org/forums/).

Keep in mind that using SFML is not a very suitable way to learn the bare basics of C++ programming, and as such it is recommended that any questions regarding general C++ be asked in more adequate forums where people proficient in C++ can help you better.

* A basic C++ tutorial can be found at (http://www.cplusplus.com/doc/tutorial/).
* An online C++ reference can be found at (http://www.cplusplus.com/reference/).

Addtionally you also find people in the [inofficial IRC chat](http://en.sfml-dev.org/forums/index.php?topic=2997.0).

## <a name="build-use"/>Building and Using SFML

### <a name="build-build"/>How do I build SFML?

Laurent has provided tutorials with each version of SFML. The first part of these tutorials is aimed at getting started, which includes building SFML with CMake and your build tool of choice, as well as setting up your IDE (if you use one) for use with SFML.

* [1.6 tutorials](http://sfml-dev.org/tutorials/1.6/)
* [2.0 tutorials](http://sfml-dev.org/tutorials/2.0/)

### <a name="build-nightly"/>Are there any nightly builds?

There are no official nightly builds, however there is a thread on the forum where unofficial nightly builds are provided for certain platforms.

[Link to the thread](http://en.sfml-dev.org/forums/index.php?topic=9513.0)

### <a name="build-environment"/>How do I setup my development environment to work with SFML?

This is covered quite thoroughly in the tutorials section for some of the most popular IDEs.

Check out the Getting Started sections of the [tutorials](http://sfml-dev.org/tutorials/).

### <a name="build-fuse"/>I want to fuse the libraries into one. (Not recommended)

To fuse two libraries, you can use the ar.exe utility provided with MinGW. You'll also need a minimal Unix environment (like [CYGWIN](http://www.cygwin.com/)). The syntax is:

    ar xv lib1.a | cut -f3 -d ' ' | xargs ar rvs lib2.a

Here are the commands to together the external dependencies:

    ar xv libgdi32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s-d.a && rm *.o && echo 'done'
    ar xv libgdi32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s.a && rm *.o && echo 'done'
    ar xv libopengl32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s-d.a && rm *.o && echo 'done'
    ar xv libopengl32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s.a && rm *.o && echo 'done'
    ar xv libwinmm.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s-d.a && rm *.o && echo 'done'
    ar xv libwinmm.a | cut -f3 -d ' ' | xargs ar rvs libsfml-window-s.a && rm *.o && echo 'done'
    ar xv libws2_32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-network-s-d.a && rm *.o && echo 'done'
    ar xv libws2_32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-network-s.a && rm *.o && echo 'done'
    ar xv libfreetype.a | cut -f3 -d ' ' | xargs ar rvs libsfml-graphics-s-d.a && rm *.o && echo 'done'
    ar xv libfreetype.a | cut -f3 -d ' ' | xargs ar rvs libsfml-graphics-s.a && rm *.o && echo 'done'
    ar xv libopenal32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s-d.a && rm *.o && echo 'done'
    ar xv libopenal32.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s.a && rm *.o && echo 'done'
    ar xv libsndfile.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s-d.a && rm *.o && echo 'done'
    ar xv libsndfile.a | cut -f3 -d ' ' | xargs ar rvs libsfml-audio-s.a && rm *.o && echo 'done'

### <a name="build-link"/>What and how do I link to use SFML?

When you want to use SFML, you need to link to the library files that provide the functionality you make use of in your application.

SFML is divided into 5 modules:

* **System** _provided by sfml-system_
* **Window** _provided by sfml-window_
* **Graphics** _provided by sfml-graphics_
* **Audio** _provided by sfml-audio_
* **Network** _provided by sfml-network_

Be aware that the modules have interdependencies on each other. For instance, if you plan on using the Graphics module, you will also have to link against the Window and System modules as well.

Dependencies:

* **System** does not depend on anything and can be used by itself.
* **Window** depends on **System**.
* **Graphics** depends on **Window** and **System**.
* **Audio** depends on **System**.
* **Network** depends on **System**.

As you can see you will always have to link against sfml-system, no matter what you do.

Be aware that some linkers are sensitive to the order in which you specify libraries to link against.

GCC (which implies MinGW as well) requires that the dependees (libraries that others depend on) are specified after the dependers (libraries that depend on others).

An example of a GCC command line linking all modules would be as follows:
```
g++ main.o -o sfml-app -lsfml-graphics -lsfml-window -lsfml-audio -lsfml-network -lsfml-system
```

This is explained as well in [this forum post](http://en.sfml-dev.org/forums/index.php?topic=8518.msg57257#msg57257).

In Code::Blocks, you would have to make sure the dependees are under the dependers in the list of libraries to link against.

## <a name="graphics"/>SFML Graphics

### <a name="graphics-image-formats"/>What image formats does SFML support?

SFML can load the following file formats: bmp, dds, jpg, png, tga, psd
But keep in mind that not all variants of each format are supported.

### <a name="graphics-white-rect"/>Why do I get a white/black rectangle instead of my texture?

This is due to a premature destruction of the sf::Texture. An sf::Sprite only holds a reference to the sf::Texture bound to it. You have to keep the sf::Texture “alive” as long as the sprite uses it. It can also be that you never bound a texture to the sprite, hence you need to call `setTexture()` with the initial texture it is to use.

When your texture is relocated from one memory location to another you have to inform your sprite of this using its `setTexture()` function as well.

### <a name="graphics-image-texture"/>What is the difference between sf::Image and sf::Texture?

In essence, there is no difference between the 2 data structures. The main question you have to ask yourself is not what the difference is, but where the data is stored.

In the case of sf::Image, the image data is stored in client-side memory, meaning in your system RAM. In there it is just an array of bytes (4 per pixel) that make up the image you can see on your screen.

In the case of sf::Texture, it is also image data, however this data resides in server-side memory, meaning in your graphics RAM next to your GPU. This memory is managed completely by OpenGL and the sf::Texture is merely a handle to that block of memory in graphics RAM. There are many forms of storage of textures but SFML uses the same layout for sf::Texture as it does for sf::Image namely quad-byte RGBA.

You can convert freely between sf::Image and sf::Texture, however just keep in mind that the bus bandwidth is limited and doing this too often means you are transferring a lot of data between graphics RAM and system RAM leading to a slowdown of all graphics related operations.

### <a name="graphics-sprites"/>My FPS count drops when drawing a lot of sprites.

This may happen due to many reasons, some may depend on the code and circumstances, but a possible bottle-neck is the amount of draw calls. When using a lot of sf::Sprites you need to make an equal amount of draw calls to make them show. 

As each draw call changes the RenderTarget it can become slow to modify it in n-amount of loops and calls when it would be most efficient to do it in just one, the reason SFML doesn't work like this is that it would limit what you can do with sf::Sprite and would force the usage of a single texture in cases where this might not be desirable or even possible due to GPU limitations. 

In order to raise FPS in this circumstance there is no choice other than using bare sf::Vertex/sf::VertexArray or finding another way to achieve the same effect. These on their own however won't be magical and a wrapper will most likely help do the desired render in a nice looking notation.

In the wiki's source codes you can find some classes used for fast rendering in some cases such as Tile-Map renderers or containers for transformable sprites that use the same texture.

### <a name="graphics-vsync-framelimit"/>Should I use VSync, window.setFramerateLimit or something else?

### <a name="graphics-xsprite"/>Should I use one sprite or x sprites to draw x textures?

Generally, the less objects you have to draw, the faster your application will run. This is almost always true. If you can, you should try to group things that are always drawn together and draw them using a single sprite. This will save you a lot of additional processing time.

What you should not do is only use a single sf::Sprite and keep binding different sf::Textures to it. Although you are only using a single sf::Sprite, you are still creating as many draw calls as if you would have an sf::Sprite for each sf::Texture. The overhead of rebinding an sf::Texture to the sf::Sprite many times during a frame will noticeably affect your performance.

You can also try to perform rudimentary culling. Culling consists of not drawing things that you know cannot be seen anyway because they are entirely covered by something else. If this can be done in your application and you prevent a lot of unneeded draw calls, you will gain performance.

### <a name="graphics-bounds"/>What is the difference between LocalBounds and GlobalBounds?

### <a name="graphics-low-fps"/>My FPS is very low when running my application.

### <a name="graphics-broken-rendertexture"/>RenderTextures don't work on my computer!

## <a name="audio"/>SFML Audio

### <a name="audio-formats"/>What audio formats does SFML support?

In addition to the formats supported by [libsndfile](http://www.mega-nerd.com/libsndfile/#Features) (wav, flac, aiff, au, raw, paf, svx, nist, voc, ircam, w64, mat4, mat5 pvf, htk, sds, avr, sd2, caf, wve, mpc2k, rf64) the Audio module is also capable of playing ogg files.
Unfortunatly MP3 is covered by a license from Thompson Multimedia and thus support for it is not included in SFML. For more information regarding the MP3 license, see http://www.mp3licensing.com.

## <a name="network"/>SFML Networking

### <a name="network-create-network-app"/>How do I create &lt;insert popular application type here&gt;?

The first thing you need to do is understand how the underlying networking of said application works. Behind every complex looking application there is a simple system of how systems send data to each other and what kind of data they send.

There are 2 kinds of topologies:

* Client-Server
* Client-Client (Peer-to-Peer)

Client-Server is generally easier to set up and often achieves higher performance than Client-Client due to the fact that dedicated servers are already configured to handle a large amount of traffic from multiple connected systems. When running a server application you can also be sure that clients can not manipulate the game state (cheat) themselves unless they have direct access to the server and can execute things there (which requires logging in etc.).

When running in a Client-Client configuration, the first thing to overcome is the initial connection establishment. Home/Office gateways/routers are mostly configured by default to not accept any incoming connection requests. As such none of the sides can establish a connection to the other. There is a way of overcoming this called [NAT hole punching](http://en.wikipedia.org/wiki/NAT_hole_punching), however this is beyond the scope of this FAQ. Care has to be taken to ensure that nobody is able to cheat in this topology. This is usually done by mirroring the game state across all involved systems and performing checks and synchronization at every game step.

Once you have picked a suitable topology for your application, you can start to think about what kind of data you want to send between the systems. There is no general answer or recommendation for this as this is very application specific and you will have to rely on good judgement to make the right choices.

### <a name="network-tcp-vs-udp"/>Should I use TCP or UDP sockets?

One must first understand what the difference is between TCP and UDP.

TCP is connection based and provides **reliable and ordered delivery of data** from one endpoint to the other. Connection based means you need to establish a TCP connection before you can make use of it for data transfer. It takes care of everything you can possibly think of (and might not think of) except creating and using the data that it sends and receives, that is still your task. Simply put, when you have an established TCP connection, whatever you shove into one end will in almost all cases come out the other end, in the right order, even over very very bad internet connections. If it doesn't arrive at the other end, both sides will be notified of the error and can perform the necessary error recovery. This comes at a price however, TCP packet headers are **at least 20 bytes and at most 60 bytes** large. This means that if you send a packet containing 1 byte over a TCP connection, your network adapter will need to transmit at least 21 bytes to get the data to the other end (disregarding the overhead from the lower OSI layers which is always present both in TCP and UDP). Also note that TCP streams are bi-directional, meaning you can use them to send and receive data in both directions, there is no need to create 2 separate TCP connections to transfer data between 2 endpoints (unless the data streams have different purposes and you want to separate them at the transport layer).

UDP provides **unreliable delivery of data** from one endpoint to the other. It is not connection based and **does not guarantee anything**. This means that if you are unlucky, you might not receive what you send, what you send might be received in a different order than the order you sent in or you might even receive duplicates of what you sent. In case of a transfer failure UDP also won't notify you that anything happened, it is completely up to you to take care of detecting and handling all these cases. The advantage of UDP is that its header size is always 8 bytes (40% of TCP) so if you send very small packets very often, you can save a lot of bandwidth if you use UDP. At larger packet sizes the header overhead is amortized and both protocols are just as efficient bandwidth-wise as the other. In the case of UDP, because there is no concept of a connection, it is your responsibility to keep track of who is sending and receiving what data because in general a server will only bind a single UDP socket with which all clients will send data to (unless they open up a new socket per client). You can also send data to multiple clients over one UDP socket because you always have to specify the destination of a datagram.

You might be wondering: "Woah! Who would even use UDP instead of TCP? There are so many disadvantages."

Well... If you implement your own reliable transport protocol on top of UDP, you can consider it as a form of "fine-tuned TCP" that does exactly what you want and no more. You could reduce the communication overhead considerably and even have the advantage that UDP traffic generally gets processed faster when travelling over the internet, which means lower latency. This is the reason why many latency critical applications choose to use UDP instead of TCP.

If you are developing your first networked application, stick with TCP as long as you can. Bandwidth and latency issues will not be your biggest concern until you are sure you can make money off it. Once that time comes, you will have gained so much experience that the decision will be trivial.

### <a name="network-blocking-non-blocking"/>Should I use blocking or non-blocking sockets?

### <a name="network-selectors"/>How do selectors work?

### <a name="network-internet-network"/>I can't connect to the other computer over the internet!

## <a name="window"/>SFML Window

### <a name="window-transparent-window"/>How do I make my window transparent?

Unfortunately SFML can't help you with this. The style and representation of the application window within your window manager/desktop environment is specific to the environment you are currently in. Because of this SFML cannot provide a uniform interface for controlling the representation of your application window.

SFML does however provide sf::Window::getSystemHandle(). Using the handle you can do a bit of research and find out how to manipulate the window representation yourself using the functions of your window manager.

### <a name="window-get-frame-time"/>What happened to getFrameTime()?

getFrameTime() was removed from SFML at the beginning of 2012. The reasoning for it can be found here: http://en.sfml-dev.org/forums/index.php?topic=6831.0

Users have to now create an sf::Clock object and keep time themselves. This has more advantages than disadvantages including:

* Correct time reporting (getFrameTime() reported the time spent **during the last frame**)
* More control over between which points in your code the time is to be measured
* More control over the precision required

## <a name="system"/>SFML System

### <a name="system-unicode"/>Does SFML support Unicode?

SFML supports the input and display of international characters, via the UTF-16 encoding. Input is provided via sf::Event::TextEntered, and display via sf::String.

### <a name="system-string-convert"/>How do I convert from sf::String to &lt;type&gt; and vice-versa?

For conversions to `sf::String`, you can just construct the `sf::String` directly from whatever object you already have. 
```cpp
sf::String sfml_string( cpp_string );
```
There are enough constructors that take care of implicit conversion from all standard C++ string types. If you want to see what these look like, take a look in the [`sf::String` documentation](http://sfml-dev.org/documentation/2.0/classsf_1_1String.php). If you want to convert from a non-C++ string to `sf::String`, it is recommended to first convert to a C++ string and then to an `sf::String`. Since any library using custom string types should provide support for this, this shouldn't be problematic.

For conversions from `sf::String` to any other custom string type, it is also recommended to first convert to a C++ string then from C++ string to that type.

`sf::String` supports implicit conversion to `std::string` and `std::wstring`, so things like
```cpp
std::cout << sfml_string << std::endl;
cpp_string += sfml_string;
std::size_t pos = cpp_string.find( sfml_string );
```
are all valid. Additionally, if implicit conversion isn't something that comforts you, you can explicitly call `.toAnsiString()` or `.toWideString()` to convert to the corresponding types.

Because internally `sf::String` stores its data in a `std::basic_string<sf::Uint32>`, conversions to this type (which is a C++ string type) are lossless. Ironically however, conversion to this type is not supported directly by `sf::String` and one has to perform it via iterators as such:
```cpp
std::basic_string<sf::Uint32> basic_uint32_string( sfml_string.begin(), sfml_string.end() );
```
Once you have a `std::basic_string<sf::Uint32>` you can use all of the same functionality as `std::string` since `std::string` is just a typedef for `std::basic_string<char>`.

### <a name="system-threads-crash"/>My program keeps crashing when I use threads!

### <a name="system-mutex"/>How do I use sf::Mutex?

### <a name="system-thread-container"/>Why can't I store my sf::Thread in an STL container?

sf::Thread inherits from sf::NonCopyable meaning you cannot copy or assign an sf::Thread. This is however a requirement to use a data type with an STL container.

You might be wondering how it would still be possible to store multiple threads in an STL container if you are trying to implement some sort of thread pool. The answer is simple: instead of storing instances, store pointers to the sf::Threads.

The sf::NonCopyable restrictions can only apply to instances of an sf::Thread. When copying pointers, the copy constructor or assignment operator of a class are not invoked. As such it is perfectly legal to copy and pass pointers around.

Since working with raw pointers is something you want to avoid in modern C++, you can use [smart pointers](#prog-smart) to great extent in combination with this technique.

## <a name="programming"/>Programming in General

### <a name="prog-raii"/>What is RAII and why does it rock?

RAII is an acronym for **R**esource **A**cquisition **I**s **I**nitialization. It is a programming idiom/technique that is used extensively in C++ and can be used in other programming languages as well.

The origin of this idiom lies in the way of how exceptions work in C++. When an exception is thrown in a code block, execution is diverted to the relevant handlers and the program flow continues on from there. If the programmer has to rely on ALL his code in a block being executed to perform the necessary clean up of resources, this can be a problem in the case an exception is thrown because it is not guaranteed that the clean up code will be executed and resources will not be destroyed properly.

To address this issue, RAII takes care of the fact that although not all code is executed when an exception is thrown, the destructors of an object are guaranteed to be called because all objects have to be destroyed when the corresponding scope is left.

This can be used in any resource owning object or in places where certain functions have to be called to make sure the program stays in a clean running state. A good example of this is an sf::Texture. No matter what happens, if the sf::Texture object gets destroyed because it goes out of scope, the underlying OpenGL resources also get freed.

From the example just mentioned, it should be apparent that the same can be said of dynamically allocated memory. It is also a resource that we need to free when we are done with it. Because of this, to promote RAII in modern C++, smart pointer facilities have made their way into the language, either as an extension or in C++11 as a part of the STL. Any memory governed by them is guaranteed to be released when they go out of scope (in the case of non-shared ownership e.g. scoped_ptr) due to RAII. This makes it much easier to use dynamically allocated objects, as you do not have to worry too much about memory management anymore.

### <a name="prog-mmm"/>Why shouldn't I manually manage my memory?

Nobody forces you to use automatic memory management, however in practice, the larger and more complex a project gets, the harder it is to keep track of such things as well. Generally it is a good idea to use automatic memory management because it frees you to dedicate more time to more important parts of your project. Not only that, it will be easier to debug and leaks will be nearly impossible.

C++ has come a long way and in its latest incarnation, C++11, many new memory management facilities made it into the C++ STL. This lets you use these constructs without the need for external or self-written libraries.

### <a name="prog-smart"/>What are smart pointers?

Smart pointers, as their name implies are pointers that are... well... smart :). Regular "raw pointers" are just values, merely an address in memory where an object is located. Unless you do something with those values, nothing is going to happen with the object, and it will sit there until you choose to free the corresponding memory block or close the program and destroy the virtual memory pages associated with it.

Smart pointers on the other hand, do things with the values that they contain. They are no longer just values but objects that govern the memory they point to. This can help you to track how many pointers exist that point to a certain block of memory (shared_ptr) or to prevent you from having multiple pointers point at the same block of memory at the same time (unique_ptr). The best part is, if you let these smart pointers manage your memory for you, they will also take care of freeing it when they think it isn't used any more.

### <a name="prog-global"/>Why shouldn't I use global variables?

Usage of global variables is considered as bad programming practice. They might seem like an easy solution to your initial problem but they will become a headache later on when the project gets bigger or you are unaware of the implications of declaring something in global scope.

One of the most dangerous things of declaring non-POD ([plain old data](http://en.wikipedia.org/wiki/Plain_old_data_structure)) objects in global scope is that you can never be sure when they are actually constructed and when they will be destroyed. This means that if they need to own resources you need to make sure they are available before the object is created, which can be tricky to do if that takes place before your main() code gets executed. Analogous to that, the object might get destroyed after your main() returns thus leaving resource destruction up to some self-clean-up mechanism or in the worst case to a resource manager that already got destroyed before main() returned. This leads to leaks and is very bad practice. Furthermore the initialization order and destruction order is not well-defined. It is only defined _within one translation unit_ as being dependant on the order of declaration, however you can't count on global variables from different translation units being constructed or destroyed in a specific order, it is pure luck here.

Another problem with global variables is that sooner or later you are going to have so many of them that they will clog up your namespace. Unless you declare them in a separate namespace they will all be in the same giant one: the global one. If you happen to declare a local variable in one of your functions that happens to have the same name as the global one you are actually referring to, you will not notice the global variable get shadowed unless you have certain warnings switched on. Some people suggest using Hungarian notation to solve this problem but the modern demeanour tends to avoid Hungarian notation as well.

There really isn't any reason to use global variables. Code using global variables can always be written without them and will never perform worse, most of the time performing better as a result of better memory usage.

### <a name="prog-insteadof-global"/>What should I use then instead of global variables?

You should try to confine variables to the scope(s) where they are used. Construct and hold them in the object which is supposed to own them as well as manage their lifetime and pass them to other objects/functions using references/pointers. Avoid passing by value. Often you cannot copy objects and are thus not allowed to pass by value, other times copying the object can take much more time because temporary memory has to be allocated just for the call and freed after the function returns.

If you happen to use a C++11 compliant compiler then you can be sure that many Standard Library objects you pass around will have move constructors defined which makes it less painful to "pass by value" since if certain conditions are met, the compiler will use the Rvalue reference version of certain functions to speed up execution by a substantial amount.

### <a name="prog-singleton"/>Why is the singleton pattern not a good one?

## <a name="trouble"/>Troubleshooting

### <a name="tr-grl"/>General

### <a name="tr-grl-trouble"/>I'm having trouble using SFML.

First, make sure that you have followed the installation instructions in the [official tutorials](http://www.sfml-dev.org/tutorials/).

Have you:

* Provided the path to the SFML headers to your compiler?
* Provided your text editor with the path to the SFML library?
* Included the headers for the packages you're using? (“SFML/[capitalized name of module].hpp”)
* Linked with the packages you're using? (See the dependencies section of this document)
* On Windows, have you copied the libsndfile-1.dll and openal32.dll files (you can find them in the complete SDK) into the folder for executable, along with the DLLs for the packages you're using (and all of their dependencies)?
* On Linux, have you installed the libraries (sudo make install in the SFML folder)?

If you've checked all of those, and SFML still refuses to work, see [I found a bug!](#grl-bug).

### <a name="tr-grl-undefined-ref"/>I keep getting "undefined reference to &lt;some strange thing that looks like an SFML function&gt;" errors!

See [What and how do I link to use SFML?](#build-link)

### <a name="tr-grl-crash"/>My computer crashes when I run my SFML program!

SFML was designed in a way that should not cause your computer to crash/freeze/hang/BSOD in any way. If it does exhibit such behaviour specifically when running your SFML program, it might be only indirectly because of SFML.

If you are using unstable drivers (still in beta or off a development branch in your package manager) chances are that they are the root cause of the problem. The reason why they cause more problems with SFML than with other programs/games is mainly because OpenGL related bugs in drivers are given low priority over DirectX related bugs simply because the latter affect more games. You'll just have to be patient and wait for the relevant bug to get fixed or revert to an older, more stable driver.

If you are not using unstable drivers, crashing might still be caused by overclocking (either self-performed or automatic) of your hardware. Try running everything at standard performance settings and see if that fixes the problem. If your hardware comes overclocked by default, search around the internet for other people who might be experiencing the same problems. If this is the case you should contact your retailer/card manufacturer as this is a hardware related issue and you won't be able to do much on your own.

### <a name="tr-grl-bug"/>I found a bug!

Most of the time any unexpected behaviour is a result of misunderstanding how to use SFML. Out of many bug reports only few of them turn out to be real bugs **which are caused by SFML itself and nothing else**.

If you think you have found a bug and are still using SFML 1.6, note that support for 1.6 had ceased long ago. It is highly recommended to upgrade to 2.0. Any bug reports made for SFML 1.6 will be ignored unless they were carried over to 2.0 as well, however this is very unlikely. If you are using 2.0, try using the latest [nightly build](http://en.sfml-dev.org/forums/index.php?topic=9513.0). There are many things that have already been fixed between the RC which is available on the site and the latest development version.

If the bug is still present in the latest SFML version, try to produce a [minimal compilable code example](#tr-grl-minimal) that displays the bug and nothing else. That way the developers and others can focus on why it is occurring.

If you can reproduce what you think is a bug, if you have another computer at your disposal, try to run it there as well. If the bug does not occur there, try to reconfigure the corresponding hardware/software settings on the first PC. A lot of strange behaviour is a result of misconfigured/faulty software/drivers. **WARNING: Trying to report a bug that is a result of the usage of beta drivers is not a good idea. The source of the problem does not lie within the responsibility of the SFML developers and as such they can't do much to fix it themselves.**

When you are sure that the bug is a result of SFML internals and is platform independent, you can go ahead and post in the forum of the package in question, and don't forget to provide a precise description of your problem, the version of SFML you're using, your system configuration, and the compilable code, and if the situation requires, the logs of your compiler and/or linker.
Also make sure that the bug hasn't already been reported (use the [search function](http://en.sfml-dev.org/forums/index.php?action=search)), confirmed (look on the [issue tracker](https://github.com/LaurentGomila/SFML/issues?page=1&state=open) or even resolved in the latest source (check also the [closed issues](https://github.com/LaurentGomila/SFML/issues?page=1&state=closed)).

### <a name="tr-grl-minimal"/>What is a minimal code?

A minimal code example is a snippet of source code that is compilable with very little effort.

This implies that:
* it consists of a single file (no extra .h, .hpp or .cpp files)
* there are no special compiler/linker options that need to be set
* the provided code is specific to the matter at hand and does not do more than it needs to

Such an example should never have to be longer than 40 lines of code (including the header include lines which of course have to be provided as well) unless it only happens in very specific cases.

An example of minimal code:
```cpp
#include <SFML/Graphics.hpp>

int main() {
	sf::RenderWindow window( sf::VideoMode(640, 480, 32), "Bug" );
	
	// Some bug specific code here...
	
	while( window.isOpen() ) {
		sf::Event event;
		while( window.pollEvent( event ) ) {
			if( event.type == sf::Event::Closed ) {
				window.close();
			}
		}
		
		// ... and here.
		
		window.display();
	}

	return EXIT_SUCCESS;
}
```

See also [the rules](http://en.sfml-dev.org/forums/index.php?topic=5559.msg36368#msg36368) for further details.

### <a name="tr-grl-obtain-minimal"/>And how can I easily obtain this minimal code?

Easy :

* Create a separate sandbox project with a single file consisting of a `main()` function
* Copy and paste your code into the body of the `main()` in such a way that it can compile
* One by one delete all lines which are not relevant to the actual problem and try to compile after every deletion to see if the problem still persists

Side note: This technique will often help you troubleshoot the problem on your own.

---

### <a name="tr-cb"/>Code::Blocks

### <a name="tr-cb-static-linker"/>I've recompiled the static version of SFML and I'm getting linker errors.

MinGW does not include the external libraries when doing static compilation, so you must add them after compilation, or link them directly into your project.

### <a name="tr-cb-linker"/>I'm getting linker errors.

With MinGW, you must link the libraries in a precise order: if libX depends on libY, libX MUST be linked before libY. For example, if you use the Graphics and Audio modules, the correct linking order would be the following: sfml-audio, sfml-graphics, sfml-window, sfml-system.

If you use the dynamic versions of the SFML 1.6 libraries, you must also define the SFML_DYNAMIC symbol in the options for your project.
If you use the static verions of the SFML 2.0 libraries, you must also define the SFML_STATIC symbol in the options for your project.
For more details, see the installation tutorial for Code::Blocks.

---

### <a name="tr-vs"/>Visual Studio

### <a name="tr-vs-crash"/>My project crashes randomly, but I don't get any compiler or linker errors.

Make sure that you're linking against the correct version of the libraries for your project. If you're compiling in Debug mode, you must link with the Debug versions of SFML, and vice-versa for Release mode. To recall, the naming conventions for SFML are:

* sfml-[module].lib for the Release DLL
* sfml-[module]-d.lib for the Debug DLL
* sfml-[module]-s.lib for the static Release DLL
* sfml-[module]-s-d.lib for the static Debug DLL.

If you link with the DLL versions, you must copy the required DLLs beside your executable:

* sfml-[module].dll for the Release DLL
* sfml-[module]-d.dll for the Debug DLL

---

### <a name="tr-win"/>Windows

### <a name="tr-win-console"/>Why does a console attach itself to my project?

In Windows, if you compile your project, you will have a console that attaches itself behind your window. To avoid this, you must create a the correct type of project or change its type after creation. Many IDEs have options allowing you to choose whether you want to create a console or a GUI application. It is however recommended to create an empty project and manually set its type afterwards. This ensures that nothing else is set automatically that might cause strange behaviour later on.

If you have already created the console project or created an empty one:

* In Code::Blocks, open the project options (Project Menu -> Properties). In the Build targets tab, select the build target you wish to change on the left (most of the time only Debug and Release exist) and change its type option in the drop-down list on the right side from "Console application" to "GUI application".
* In Visual Studio, go to the project options (Project Menu -> Properties). In the tree on the left, expand the "Configuration properties" tree and expand the "Linker" sub-tree. Select "System" from the sub-tree, and in the SubSystem field on the right side change "Console (/SUBSYSTEM:CONSOLE)" to "Windows (/SUBSYSTEM:WINDOWS)" by clicking on the field and using the drop-down list.

To maintain a portable entry point (`int main()` function), you can link your program against the small sfml-main.lib library in the case of Visual Studio or libsfml-main.a in the case of Code::Blocks/MinGW.

Alternatively to hide the console, you can also define your own Windows entry point for graphical applications.

```cpp
int WINAPI WinMain(HINSTANCE hThisInstance, HINSTANCE hPrevInstance, LPSTR lpszArgument, int nCmdShow)
```

Replace your `int main()` or `int main(int argc, char** argv)` with this function and it will be called by the operating system when your program is executed just like the classical `int main()` function.

---

### <a name="tr-lnx"/>Linux

### <a name="tr-lnx-compile"/>(Debian) I can't compile the source code.

Before anything else, make sure that you've followed the [official tutorial](http://www.sfml-dev.org/tutorials/) and then check if the following packages have been installed:

* libgl1-mesa-dev
* libglu1-mesa-dev
* libopenal-dev
* libopenal1-dbg
* libsndfile1-dev
* libx11-dev
* libx11-6-dbg
* libfreetype6-dev
* libxrandr-dev
* libxrandr2-dbg

If you're compiling version 1.3, you must add
```cpp
    #include <string.h>
```
in Packet.hpp in the Network module, and SoundFileDefault.cpp in Audio.

If you're using the git version, remember to do a make clean before compilation to avoid any problems.

### <a name="tr-lnx-titlebar"/>There is no titlebar visible and/or artifacts from windows are visible.

If you're running compiz, then turn it off, because compiz interfere with other OpenGL applications.

You can use this simple script to toggle compiz on and off, if you're using metacity as your window manager:

```bash
    #!/bin/bash
    pid=`ps --no-heading -C compiz.real | cut -d "?" -f1`;
    if [ -n "$pid" ]; then
      metacity --replace &
    else
      compiz --replace &
    fi
```

## <a name="licensing"/>Licensing

### <a name="lic-license"/>What license does SFML have?

SFML is under the [zlib/png license](http://www.opensource.org/licenses/zlib-license.php). You can use SFML for both open-source and proprietary project, including paid or commercial ones. If you use SFML in your projects, a credit or mention is appreciated, but is not required.

### <a name="lic-commercial"/>Can I use SFML in commercial applications?

Yes, you may use SFML in commercial applications. You don't even have to mention that you used SFML in your application, but the zlib license states that if you do mention it, you are not allowed to state that you yourself are the author of SFML. You are also not allowed to modify SFML and represent it as being the original.

### <a name="lic-static"/>Can I link SFML statically?

Yes, you can link SFML statically. This can be done in any operating system, although in Linux and Mac OS X it is recommended to only link dynamically unless you have special requirements.

When linking statically, do not forget to specify the SFML_STATIC define on your command line.

### <a name="lic-examples"/>Can I use the code from the example directory?

The code from the example directory is not marked as being provided under a separate license. As such it is also governed by the zlib/png license and you are free to do anything you want with the code as long as it complies to the license.

### <a name="lic-pay"/>Do I have to pay any license fees or royalties?

The zlib/png license is a permissive free software license which means it has no provisions to monetize the software it covers. As such SFML can be used for free with no requirement to pay any fees or royalties to its author.

## <a name="libraries"/>Libraries for SFML

### <a name="libraries-gui-package"/>Does SFML have a GUI package?

No, SFML does not have a GUI package, but you can essentially use any OpenGL-based GUI library. Here's a obviously incomplete list:

* [SFGUI](http://sfgui.sfml-dev.de/)
* [GWEN](https://github.com/garrynewman/GWEN)
* [TGUI](http://tgui.weebly.com/)
* [CEGUI](http://www.cegui.org.uk/wiki/index.php/Main_Page)
* [Guichan](https://code.google.com/p/guichan/)
* [libRocket](http://librocket.com/)

### <a name="libraries-gui-lib"/>Can you interface SFML with a GUI library?

Yes, you can! See examples for Qt, wxWidgets, and the native Win32 and X11 APIs in the [official tutorials](http://www.sfml-dev.org/tutorials/).

### <a name="libraries-video"/>Can I read video files with SFML?

SFML does not have a video playback module, but one can easily connect FFMPEG or similar libraries with SFML. There even exists a maintained project from a SFML user called [sfeMovie](http://lucas.soltic.perso.luminy.univmed.fr/sfeMovie/).

### <a name="libraries-thor"/>What exactly is Thor?

Thor is an open-source and cross-platform library written in the programming language C++. It is an extension to SFML and comes with high-level features that base on SFML and that are supposed to help in daily C++ routine, especially with respect to graphics and game programming.

The git repository is also hosted on [GitHub](https://github.com/Bromeon/Thor) and the official website can be found at: http://www.bromeon.ch/libraries/thor/

## <a name="misc"/>Miscellaneous

### <a name="misc-projects"/>Are there any famous projects with SFML?

As SFML is open source and has a permissive free software license game creators are not force to specify that they've used SFML in their games, thus it might well be that SFML has been used in bigger commercial projects. As for the known projects, there's a [dedicated page](Projects) on the SFML's wiki.

### <a name="misc-distr"/>How can I distribute my game?

### <a name="misc-upload"/>Where can I upload my game to?

There are many possible places to share your game and game related data with the world wide web and their users. For SFML someone as created a website explicitly for sharing content related to SFML: http://www.sfmluploads.org/

Another worth mentioning provider is [MediaFire](http://www.mediafire.com/). It's a simple and fast file sharing hoster.

### <a name="misc-examples"/>Are there any example projects I can learn from?

Examples of the usage of each module are provided with SFML itself if you download the full SDK version. If you want full projects that make use of many features of SFML at the same time your best bet is to check out the [projects forum](http://en.sfml-dev.org/forums/index.php?board=10.0). People frequently showcase what they are working on there for the community to see. Often they might provide the source code as well, so if you feel brave enough you can look in there if you find a certain project interesting.
