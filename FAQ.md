# Frequently asked questions (FAQ)

- **[Main](#main_)**
  - [What is SFML?](#whatis)
  - [On which platforms is SFML currently available?](#platforms)
  - [In which languages is SFML available?](#languages)
  - [Under what license is SFML available?](#license)
  - [What dependencies does SFML have?](#dependencies)
  - [Where can I ask questions?](#questions)
  - [I found a bug!](#bug)
  - [What is a minimal code?](#minimal)
  - [And how can I easily obtain this minimal code?](#obtain_minimal)
  - [I want to propose a new feature!](#feature_request)
- [**Using SFML**](#using_sfml)
  - [Does SFML have a GUI package?](#gui_package)
  - [Can you interface SFML with a GUI library?](#embed_gui)
  - [Can I read video files with SFML?](#video)
  - [What audio formats does SFML support?](#audio_formats)
  - [What image formats does SFML support?](#image_formats)
  - [Does SFML support Unicode?](#unicode)
  - [I'm having trouble using SFML.](#trouble)
  - [I want to fuse the libraries into one.](#fuse_libraries)
- [**Troubleshooting**](#troubleshooting)
  - [(CodeBlocks / Windows) I've recompiled the static version of SFML and I'm getting linker errors.](#cb_win_static)
  - [(CodeBlocks) I'm getting linker errors.](#cb_linker)
  - [(Windows) Why does a console attach itself to my project?](#win_console)
  - [My sprite isn't displayed but a white square!](#sprite_white)
  - [(Windows / Visual Studio) My project crashes randomly, but I don't get any compiler or linker errors.](#win_vs_crash)
  - [(Debian Linux Debian) I can't compile the source code.](#deb_linux_compile)
  - [(Linux) There is no titlebar visible and/or artifacts from windows are visible.](#linux_titlebar)
  

## <a name="main_"/>Main

### <a name="whatis"/>What is SFML?

SFML is a simple to use and portable API, written in C++. You can think of it as an object oriented SDL. SFML is made of modules in order to be as useful as possible for everyone. You can use SFML as a minimalist window system in order to use OpenGL, or as a complete multimedia library full of features to build video games or multimedia softwares.

You can find a more specific presentation of its features on [this page](http://www.sfml-dev.org/features.php).

### <a name="platforms"/>On which platforms is SFML currently available?

SFML is currently available and fully functional in Windows (8, 7, Vista, XP, 2000, 98), Linux and Mac OS X. SFML works on both 32 and 64 bit systems.

### <a name="languages"/>In which languages is SFML available?

SFML is implemented in C++. That said, several bindings have been created for other languages that allow SFML to be used from C, C#, C++/CLI, D, Ruby, OCaml, Java, Python and VB.NET.

### <a name="license"/>Under what license is SFML available?

SFML is under the [zlib/png license](http://www.opensource.org/licenses/zlib-license.php). You can use SFML for both open-source and proprietary project, including paid or commercial ones. If you use SFML in your projects, a credit or mention is appreciated, but is not required.

### <a name="dependencies"/>What dependencies does SFML have?

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

### <a name="questions"/>Where can I ask questions?

Post any questions in the [SFML forum](http://en.sfml-dev.org/forums/).
Addtionally you also find people in the [inofficial IRC chat](http://en.sfml-dev.org/forums/index.php?topic=2997.0).

### <a name="bug"/>I found a bug!

Post in the forum of the package in question, and don't forget to provide a precise description of your problem, the version of SFML you're using, your system configuration, and compilable code, if necessary, or the logs of your compiler or linker.
Also make sure that the bug hasn't already be reported (use the [search function](http://en.sfml-dev.org/forums/index.php?action=search)), confirmed (look on the [issue tracker](https://github.com/LaurentGomila/SFML/issues?page=1&state=open) or even resolved in the latest source (check also the [closed issues](https://github.com/LaurentGomila/SFML/issues?page=1&state=closed)).

### <a name="minimal"/>What is a minimal code example?

A minimal code example is a source code which everyone can easily compile after a simple copy-paste in a single file (no extra .h, .hpp or .cpp files) and which is made only out of source code showing the bug.
See also [the rules](http://en.sfml-dev.org/forums/index.php?topic=5559.msg36368#msg36368) for further details.

### <a name="obtain_minimal"/>And how can I easily obtain this minimal code example?

Easy :

* Create a new `main()` function with all the code
* Delete line by line which are not relevant to the actual problem and try to compile to see if the bug is always present or not

Side note: This technique will often help you discover the bug on your own.

### <a name="feature_request"/>I want to propose a new feature!

Before anything else, check the (road-map)[https://github.com/LaurentGomila/SFML/issues/milestones] to see if the functionality has already been planned. If not, there is a (forum section)[http://en.sfml-dev.org/forums/index.php?board=2.0] dedicated to feature requests. Please search before posting, and stick to the spirit of SFML as a multimedia and multi-platform library. So for example a XML parser, a database library or a platform-specific function is unlikely to be accepted.

## <a name="using_sfml"/>Using SFML

### <a name="gui_package"/>Does SFML have a GUI package?

At the moment, SFML does not have a GUI package. You can certainly use any external OpenGL-based libraries, such as (SFGUI)[http://sfgui.sfml-dev.de/], (CEGUI)[http://www.cegui.org.uk/wiki/index.php/Main_Page] or (Guichan)[http://guichan.sourceforge.net/].

### <a name="embed_gui"/>Can you interface SFML with a GUI library?

Yes, you can! See examples for Qt, wxWidgets, and the native Win32 and X11 APIs in the [official tutorials](http://www.sfml-dev.org/tutorials/).

### <a name="video"/>Can I read video files with SFML?

SFML does not have a video playback module, but there exists a project from a SFML user called (sfeMovie)[http://lucas.soltic.perso.luminy.univmed.fr/sfeMovie/].

### <a name="audio_formats"/>What audio formats does SFML support?

In addition to the formats supported by [libsndfile](http://www.mega-nerd.com/libsndfile/#Features) (wav, flac, aiff, au, raw, paf, svx, nist, voc, ircam, w64, mat4, mat5 pvf, htk, sds, avr, sd2, caf, wve, mpc2k, rf64) the Audio module is also capable of playing ogg files.
Unfortunatly MP3 is covered by a license from Thompson Multimedia and thus support for it is not included in SFML. For more information regarding the MP3 license, see http://www.mp3licensing.com.

### <a name="image_formats"/>What image formats does SFML support?

SFML can load the following file formats: bmp, dds, jpg, png, tga, psd
But keep in mind that not all variants of each format is supported.

### <a name="unicode"/>Does SFML support Unicode?

SFML supports the input and display of international characters, via the UTF-16 encoding. Input is provided via sf::Event::TextEntered, and display via sf::String.

### <a name="trouble"/>I'm having trouble using SFML.

First, make sure that you have followed the installation instructions in the (official tutorials)[http://www.sfml-dev.org/tutorials/].

Have you:

* Provided the path to the SFML headers to your compiler?
* Provided your text editor with the path to the SFML library?
* Included the headers for the packages you're using? (“SFML/[capitalized name of module].hpp”)
* Linked with the packages you're using? (See the dependencies section of this document)
* On Windows, have you copied the libsndfile-1.dll and openal32.dll files (you can find them in the complete SDK) into the folder for executable, along with the DLLs for the packages you're using (and all of their dependencies)?
* On Linux, have you installed the libraries (sudo make install in the SFML folder)?

If you've checked all of those, and SFML still refuses to work, see [I found a bug!](#wiki-bug).

### <a name="fuse_libraries"/>(MinGW) I want to fuse the libraries into one. (Not recommended)

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

## <a name="troubleshooting"/>Troubleshooting

### <a name="cb_win_static"/>[CodeBlocks / Windows] I've recompiled the static version of SFML and I'm getting linker errors.

MinGW does not include the external libraries when doing static compilation, so you must add them after compilation, or link them directly into your project.

### <a name="cb_linker"/>[CodeBlocks] I'm getting linker errors.

With MinGW, you must link the libraries in a precise order: if libX depends on libY, libX MUST be linked before libY. For example, if you use the Graphics and Audio modules, the correct linking order would be the following: sfml-audio, sfml-graphics, sfml-window, sfml-system.

If you use the dynamic versions of the SFML 1.6 libraries, you must also define the SFML_DYNAMIC symbol in the options for your project.
If you use the static verions of the SFML 2.0 libraries, you must also define the SFML_STATIC symbol in the options for your project.
For more details, see the installation tutorial for Code::Blocks.

### <a name="win_console"/>[Windows] Why does a console attach itself to my project?

In Windows, if you compile your project, you will have a console that attaches itself behind your window. To avoid this, you must create a Console project, and then:

* In Code::Blocks, go to the project options (Project ⇒ Properties) and in the Build targets tab, selection the GUI Application type.
* In Visual Studio, go to the project options (Project ⇒ Properties) In the tree on the left, go to the “Configuration properties/Linker/System” and, in SubSystem field, select “Windows (/SUBSYSTEM:WINDOWS)”.

To maintain a portable entry point (`main()`), you can link your program against the small sfml-main.lib library.
Optionally you can also define the entry point on your own, which is not `int main(void)` or `int main(int argc, char** argv)`, but `int WINAPI WinMain(HINSTANCE hThisInstance, HINSTANCE hPrevInstance, LPSTR lpszArgument, int nCmdShow)`.

### <a name="sprite_white"/>My sprite isn't displayed but a white square!

This is due to a premature destruction of the sf::Texture. Indeed a sf::Sprite only references the external sf::Texture. You have to keep the sf::Texture “alive” as long as the sprite uses it. It can also be that you never gave the sprite a texture, hence you need to call `sprite.setTexture()`.

When your texture is moved from one memory place to another you have to update your sprite with `setTexture()` function).

### <a name="win_vs_crash"/>[Windows / Visual Studio] My project crashes randomly, but I don't get any compiler or linker errors.

Make sure that you're linking against the correct version of the libraries for your project. If you're compiling in Debug mode, you must link with the Debug versions of SFML, and vice-versa for Release mode. To recall, the naming conventions for SFML are:

* sfml-[module].lib for the Release DLL
* sfml-[module]-d.lib for the Debug DLL
* sfml-[module]-s.lib for the static Release DLL
* sfml-[module]-s-d.lib for the static Debug DLL.

If you link with the DLL versions, you must copy the required DLLs beside your executable:

* sfml-[module].dll for the Release DLL
* sfml-[module]-d.dll for the Debug DLL

### <a name="deb_linux_compile"/>[Debian Linux Debian] I can't compile the source code.

Before anything else, make sure that you've followed the (official tutorial)[http://www.sfml-dev.org/tutorials/] and then check if the following packages have been installed:

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

### <a name="linux_titlebar"/>[Linux] There is no titlebar visible and/or artifacts from windows are visible.

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