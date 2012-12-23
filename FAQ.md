# Frequently asked questions (FAQ)
**[General](#general)**

- [What is SFML?](#grl-whatis)
- [On which platforms is SFML currently available?](#grl-platforms)
- [Which programming languages are supported by SFML?](#grl-languages)
- [What dependencies does SFML have?](#grl-dependencies)
- [What version of SFML should I use?](#grl-version)
- [Is there a complete list with all the changes from SFML 1.6 to SFML 2.x?](#grl-changes)
- [How do I build SFML?](#grl-build)
- [Are there any nightly builds?](#grl-nightly)
- [Where can I ask questions?](#grl-questions)
- [I found a bug!](#grl-bug)
- [My computer crashes when I run my SFML program!](#grl-crash)
- [I want to propose a new feature!](#grl-reqeust)

**[Licensing](#licensing)**

- [What license does SFML have?](#lic-license)
- [Can I use SFML in commercial applications?](#lic-commercial)
- [Can I link SFML statically?](#lic-static)
- [Can I use the code from the example directory?](#lic-examples)
- [Do I have to pay any license fees or royalties?](#lic-pay)

**[Using SFML](#using)**

- [How do I setup my development environment to work with SFML?](#use-environment)
- [Does SFML have a GUI package?](#use-gui-package)
- [Can you interface SFML with a GUI library?](#use-gui-lib)
- [Can I read video files with SFML?](#use-video)
- [What audio formats does SFML support?](#use-audio-formats)
- [What image formats does SFML support?](#use-image-formats)
- [Does SFML support Unicode?](#use-unicode)
- [I'm having trouble using SFML.](#use-trouble)
- [Are there any example projects I can learn from?](#use-examples)
- [I want to fuse the libraries into one. (Not recommended)](#use-fuse)

**[Programming in General](#programming)**

- [What is RAII and why does it rock?](#prog-raii)
- [Why shouldn't I manually manage my memory?](#prog-mmm)
- [What are smart pointers?](#prog-smart)
- [Why shouldn't I use global variables?](#prog-global)
- [What should I use then instead of global variables?](#prog-insteadof-global)
- [Why is the singleton pattern not a good one?](#prog-singleton)

**[Development with SFML](#development)**

- [What is the difference between LocalBounds and GlobalBounds?](#dev-bounds)
- [Why do I get a white rectangle instead of my texture?](#dev-white-rect)
- [My FPS count drops when drawing a lot of sprites, how do I get more performance?](#dev-performance)
- [What is the difference between sf::Image and sf::Texture?](#dev-image-texture)
- [Should I use VSync, window.setFramerateLimit or something else?](#dev-vsync-framelimit)
- [Should I use one sprite or x sprites to draw x textures?](#dev-xsprite)

**[Troubleshooting](#trouble)**

- [General](#tr-grl)
 - [What is a minimal code?](#tr-grl-minimal)
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

**[Others](#others)**

- [Are there any famous projects with SFML?](#or-projects)
- [What exactly is Thor?](#or-thor)
- [How can I distribute my game?](#or-distr)
- [Where can I upload my game to?](#or-upload)

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

### <a name="grl-build"/>How do I build SFML?

Laurent has provided tutorials with each version of SFML. The first part of these tutorials is aimed at getting started, which includes building SFML with CMake and your build tool of choice, as well as setting up your IDE (if you use one) for use with SFML.

[1.6 tutorials](http://sfml-dev.org/tutorials/1.6/)
[2.0 tutorials](http://sfml-dev.org/tutorials/2.0/)

### <a name="grl-nightly"/>Are there any nightly builds?

There are no official nightly builds, however there is a thread on the forum where unofficial nightly builds are provided for certain platforms.

[Link to the thread](http://en.sfml-dev.org/forums/index.php?topic=9513.0)

### <a name="grl-questions"/>Where can I ask questions?

Post any questions in the [SFML forum](http://en.sfml-dev.org/forums/).
Addtionally you also find people in the [inofficial IRC chat](http://en.sfml-dev.org/forums/index.php?topic=2997.0).

### <a name="grl-bug"/>I found a bug!

Post in the forum of the package in question, and don't forget to provide a precise description of your problem, the version of SFML you're using, your system configuration, and compilable code, if necessary, or the logs of your compiler or linker.
Also make sure that the bug hasn't already be reported (use the [search function](http://en.sfml-dev.org/forums/index.php?action=search)), confirmed (look on the [issue tracker](https://github.com/LaurentGomila/SFML/issues?page=1&state=open) or even resolved in the latest source (check also the [closed issues](https://github.com/LaurentGomila/SFML/issues?page=1&state=closed)).

### <a name="grl-crash"/>My computer crashes when I run my SFML program!

SFML was designed in a way that should not cause your computer to crash/freeze/hang/BSOD in any way. If it does exhibit such behaviour specifically when running your SFML program, it might be only indirectly because of SFML.

If you are using unstable drivers (still in beta or off a development branch in your package manager) chances are that they are the root cause of the problem. The reason why they cause more problems with SFML than with other programs/games is mainly because OpenGL related bugs in drivers are given low priority over DirectX related bugs simply because the latter affect more games. You'll just have to be patient and wait for the relevant bug to get fixed or revert to an older, more stable driver.

If you are not using unstable drivers, crashing might still be caused by overclocking (either self-performed or automatic) of your hardware. Try running everything at standard performance settings and see if that fixes the problem. If your hardware comes overclocked by default, search around the internet for other people who might be experiencing the same problems. If this is the case you should contact your retailer/card manufacturer as this is a hardware related issue and you won't be able to do much on your own.

### <a name="grl-reqeust"/>I want to propose a new feature!

Before anything else, check the [road-map](https://github.com/LaurentGomila/SFML/issues/milestones) to see if the functionality has already been planned. If not, there is a [forum section](http://en.sfml-dev.org/forums/index.php?board=2.0) dedicated to feature requests. Please search before posting, and stick to the spirit of SFML as a multimedia and multi-platform library. So for example a XML parser, a database library or a platform-specific function is unlikely to be accepted.


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

## <a name="using"/>Using SFML

### <a name="use-environment"/>How do I setup my development environment to work with SFML?

This is covered quite thoroughly in the tutorials section for some of the most popular IDEs.

Check out the Getting Started sections of the [tutorials](http://sfml-dev.org/tutorials/).

### <a name="use-gui-package"/>Does SFML have a GUI package?

No, SFML does not have a GUI package, but you can essentially use any OpenGL-based GUI library. Here's a obviously incomplete list:

* [SFGUI](http://sfgui.sfml-dev.de/)
* [GWEN](http://code.google.com/p/gwen/)
* [TGUI](http://tgui.weebly.com/)
* [CEGUI](http://www.cegui.org.uk/wiki/index.php/Main_Page)
* [Guichan](https://code.google.com/p/guichan/)
* [libRocket](http://librocket.com/)

### <a name="use-gui-lib"/>Can you interface SFML with a GUI library?

Yes, you can! See examples for Qt, wxWidgets, and the native Win32 and X11 APIs in the [official tutorials](http://www.sfml-dev.org/tutorials/).

### <a name="use-video"/>Can I read video files with SFML?

SFML does not have a video playback module, but one can easily connect FFMPEG or similar libraries with SFML. There even exists a maintained project from a SFML user called [sfeMovie](http://lucas.soltic.perso.luminy.univmed.fr/sfeMovie/).

### <a name="use-audio-formats"/>What audio formats does SFML support?

In addition to the formats supported by [libsndfile](http://www.mega-nerd.com/libsndfile/#Features) (wav, flac, aiff, au, raw, paf, svx, nist, voc, ircam, w64, mat4, mat5 pvf, htk, sds, avr, sd2, caf, wve, mpc2k, rf64) the Audio module is also capable of playing ogg files.
Unfortunatly MP3 is covered by a license from Thompson Multimedia and thus support for it is not included in SFML. For more information regarding the MP3 license, see http://www.mp3licensing.com.

### <a name="use-image-formats"/>What image formats does SFML support?

SFML can load the following file formats: bmp, dds, jpg, png, tga, psd
But keep in mind that not all variants of each format are supported.

### <a name="use-unicode"/>Does SFML support Unicode?

SFML supports the input and display of international characters, via the UTF-16 encoding. Input is provided via sf::Event::TextEntered, and display via sf::String.

### <a name="use-trouble"/>I'm having trouble using SFML.

First, make sure that you have followed the installation instructions in the [official tutorials](http://www.sfml-dev.org/tutorials/).

Have you:

* Provided the path to the SFML headers to your compiler?
* Provided your text editor with the path to the SFML library?
* Included the headers for the packages you're using? (“SFML/[capitalized name of module].hpp”)
* Linked with the packages you're using? (See the dependencies section of this document)
* On Windows, have you copied the libsndfile-1.dll and openal32.dll files (you can find them in the complete SDK) into the folder for executable, along with the DLLs for the packages you're using (and all of their dependencies)?
* On Linux, have you installed the libraries (sudo make install in the SFML folder)?

If you've checked all of those, and SFML still refuses to work, see [I found a bug!](#grl-bug).

### <a name="use-examples"/>Are there any example projects I can learn from?

Examples of the usage of each module are provided with SFML itself if you download the full SDK version. If you want full projects that make use of many features of SFML at the same time your best bet is to check out the [projects forum](http://en.sfml-dev.org/forums/index.php?board=10.0). People frequently showcase what they are working on there for the community to see. Often they might provide the source code as well, so if you feel brave enough you can look in there if you find a certain project interesting.

### <a name="use-fuse"/>I want to fuse the libraries into one. (Not recommended)

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

## <a name="development"/>Development with SFML

### <a name="dev-bounds"/>What is the difference between LocalBounds and GlobalBounds?
### <a name="dev-white-rect"/>Why do I get a white rectangle instead of my texture?

This is due to a premature destruction of the sf::Texture. Indeed a sf::Sprite only references the external sf::Texture. You have to keep the sf::Texture “alive” as long as the sprite uses it. It can also be that you never gave the sprite a texture, hence you need to call `sprite.setTexture()`.

When your texture is moved from one memory place to another you have to update your sprite with `setTexture()` function).

### <a name="dev-performance"/>My FPS count drops when drawing a lot of sprites, how do I get more performance?
### <a name="dev-image-texture"/>What is the difference between sf::Image and sf::Texture?

In essence, there is no difference between the 2 data structures. The main question you have to ask yourself is not what the difference is, but where the data is stored.

In the case of sf::Image, the image data is stored in client-side memory, meaning in your system RAM. In there it is just an array of bytes (4 per pixel) that make up the image you can see on your screen.

In the case of sf::Texture, it is also image data, however this data resides in server-side memory, meaning in your graphics RAM next to your GPU. This memory is managed completely by OpenGL and the sf::Texture is merely a handle to that block of memory in graphics RAM. There are many forms of storage of textures but SFML uses the same layout for sf::Texture as it does for sf::Image namely quad-byte RGBA.

You can convert freely between sf::Image and sf::Texture, however just keep in mind that the bus bandwidth is limited and doing this too often means you are transferring a lot of data between graphics RAM and system RAM leading to a slowdown of all graphics related operations.

### <a name="dev-vsync-framelimit"/>Should I use VSync, window.setFramerateLimit or something else?
### <a name="dev-xsprite"/>Should I use one sprite or x sprites to draw x textures?


## <a name="trouble"/>Troubleshooting

### <a name="tr-grl"/>General
### <a name="tr-grl-minimal"/>What is a minimal code?

A minimal code example is a source code which everyone can easily compile after a simple copy-paste in a single file (no extra .h, .hpp or .cpp files) and which is made only out of source code showing the bug.
See also [the rules](http://en.sfml-dev.org/forums/index.php?topic=5559.msg36368#msg36368) for further details.

### <a name="tr-grl-obtain-minimal"/>And how can I easily obtain this minimal code?

Easy :

* Create a new `main()` function with all the code
* Delete line by line which are not relevant to the actual problem and try to compile to see if the bug is always present or not

Side note: This technique will often help you discover the bug on your own.

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

In Windows, if you compile your project, you will have a console that attaches itself behind your window. To avoid this, you must create a Console project, and then:

* In Code::Blocks, go to the project options (Project ? Properties) and in the Build targets tab, selection the GUI Application type.
* In Visual Studio, go to the project options (Project ? Properties) In the tree on the left, go to the “Configuration properties/Linker/System” and, in SubSystem field, select “Windows (/SUBSYSTEM:WINDOWS)”.

To maintain a portable entry point (`main()`), you can link your program against the small sfml-main.lib library.
Optionally you can also define the entry point on your own, which is not `int main(void)` or `int main(int argc, char** argv)`, but `int WINAPI WinMain(HINSTANCE hThisInstance, HINSTANCE hPrevInstance, LPSTR lpszArgument, int nCmdShow)`.

---

### <a name="tr-lnx"/>Linux

### <a name="tr-lnx-compile"/>(Debian I can't compile the source code.

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


## <a name="others"/>Others

### <a name="or-projects"/>Are there any famous projects with SFML?
### <a name="or-thor"/>What exactly is Thor?
### <a name="or-distr"/>How can I distribute my game?
### <a name="or-upload"/>Where can I upload my game to?