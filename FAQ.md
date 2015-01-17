### Most FAQ entries have been moved to [the official SFML FAQ](http://www.sfml-dev.org/faq.php).

# Frequently asked questions (FAQ)

**[Building and Using SFML](#build-use)**
- [I want to fuse the libraries into one. (Not recommended)](#build-fuse)

**[Troubleshooting](#trouble)**
- [General](#tr-grl)
 - [I'm having trouble using SFML.](#tr-grl-trouble)
 - [I keep getting "undefined reference to &lt;some strange thing that looks like an SFML function&gt;"!](#tr-grl-undefined-ref)
 - [Why can't I use SFML as a 64-bit library on my 64-bit system?](#tr-grl-64bit)
 - [My computer crashes when I run my SFML program!](#tr-grl-crash)
 - [I found a bug!](#i-found-a-bug)
 - [What is minimal code?](#tr-grl-minimal)
 - [And how can I easily obtain this minimal code?](#tr-grl-obtain-minimal)
- [Code::Blocks](#tr-cb)
 - [I'm getting linker errors.](#tr-cb-linker)
- [Visual Studio](#tr-vs)
 - [My project crashes randomly, but I don't get any compiler or linker errors.](#tr-vs-crash)
 - [I keep getting ```fatal error LNK1112: module machine type 'XYZ' conflicts with target machine type 'ZYX'```. Help!](#tr-vs-arch)
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

## <a name="build-use"/>Building and Using SFML

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

### <a name="tr-grl-64bit"/>Why can't I use SFML as a 64-bit library on my 64-bit system?

First of all, you have to ask yourself: Do I really need to use SFML as a 64-bit library? There are some benefits to building 64-bit applications, but it is recommended that beginners do not try this until they are confident with the compile and linking process.

Most of the confusion stems from the fact that with Windows, although modern versions make use of 64-bit processor instruction sets, they are able to run 32-bit applications as well. By default, most "standard" installations build 32-bit executables. The fact that you are running a 64-bit operating system doesn't mean that you automatically build 64-bit executables. When in doubt, download/build 32-bit executables.

Let's start with the basics. The processors that you use everyday execute instructions which is what programs are essentially made of. The set of instructions that a processor understands and is able to execute properly is known as its _instruction set_. Because Intel released a family of very popular processors quite a while ago that supported at first 16-bit registers and eventually 32-bit registers whose model numbers ended with 86, the instruction set that they provided came to be known as the x86 instruction set. Nowadays, x86 is synonymous with 32-bit, and any time you see an architecture marked as x86 you should immediately tell that it is a 32-bit architecture. Because of how addresses have to be stored in registers, eventually 32-bit registers were no longer sufficient (could only address up to 4GiB of RAM) and processors had to start providing 64-bit registers. This marked the start of the x64 era. This time around, AMD was the first to come up with the instruction set for 64-bit architectures, and thus you will hear of the term AMD64 a lot. x64, x86-64 and AMD64 all tend to refer to 64-bit architectures and are often used interchangeably.

When you compile your code, the compiler ends up translating your syntactically legal code into machine instructions to be executed on the _target CPU_. The compiler will always produce code that can run on a _specific architecture_. Because x64 architectures are fully backwards-compatible with x86 instructions (it is merely a superset), they can run executables compiled for x86 systems as well, however, the operating system kernel has to support this and allow it.

Some systems such as Linux and OS X choose to trade executable portability for performance. You will not be able to transfer binary files between systems of different architectures, but for that they will be optimized more for their target architecture. Saying that you want to "compile for 64-bit" on these systems doesn't make much sense since you will have to do so automatically anyway. There are ways of forcing compilation for 32-bit systems in order to build _somewhat_ portable binaries, but that is beyond the scope of this FAQ. Just understand that on these systems, you will almost never have to worry about choosing an architecture type.

Windows, as mentioned earlier, chose to go down a different route. They favoured portability over potential performance gains. Because Microsoft couldn't easily stop supporting legacy applications (most of them being proprietary and barely maintained) on their newer operating systems, they had to make it possible to run 32-bit applications side by side with native 64-bit applications. This technology is called _Windows on Windows_ or _Windows 32-bit on Windows 64-bit_, WoW64 in short. Every 64-bit Windows installation has a second copy of the operating system files installed as well, the 32-bit version. When running programs, the executable loader checks for the executable's architecture and loads the corresponding dynamic libraries that it needs to function. Because this information is embedded into the executable itself, it is hard to tell what architecture an executable was built for without inspecting it closer. It is perfectly normal to build for 32-bit Windows systems even though you know that all your users will be on 64-bit systems, it will simply work.

This is one of the main reasons why confusion arises when building software using libraries as well. Unless the target architecture is explicitly noted in the file name, you will not know whether the library file is compatible with the architecture type your compiler is targeting. SFML does not explicitly append an architecture type to its library files, so you will simply have to remember this after building SFML or downloading a pre-built library from somewhere.

When building with Microsoft Visual Studio, the linker checks the machine types (architectures) of all modules that are to be linked with each other. If it finds a mismatch, it will abort with an error.

The error commonly looks like this:
```
fatal error LNK1112: module machine type 'X86' conflicts with target machine type 'x64'
```
It is possible that X86 and x64 are exchanged.

To resolve this error, simply make sure that no conflicts arise. If you are targeting x64 with your executable, make sure you are only using x64 libraries as well. The same applies for x86.

If you are compiling using GCC or clang, it is less obvious when architecture conflicts arise. Instead of having explicit checks as with Visual Studio, the linkers in these toolchains merely ignore all symbols with mismatching architectures. It will look like the linker accepted the library files, but in fact it didn't process any of them at all, leading to a large number of undefined references during linking. This is especially annoying because these errors can stem from many other causes as well. It is recommended to not build for x64 on Windows if using any of these toolchains. If you really must build for x64 on Windows, use Visual Studio instead.

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
Also make sure that the bug hasn't already been reported (use the [search function](http://en.sfml-dev.org/forums/index.php?action=search)), confirmed (look on the [issue tracker](https://github.com/LaurentGomila/SFML/issues?page=1&state=open)) or even resolved in the latest source (check also the [closed issues](https://github.com/LaurentGomila/SFML/issues?page=1&state=closed)).

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

### <a name="tr-cb-linker"/>I'm getting linker errors.

With MinGW, you must link the libraries in a precise order: if libX depends on libY, libX MUST be linked before libY. For example, if you use the Graphics and Audio modules, the correct linking order would be the following: sfml-audio, sfml-graphics, sfml-window, sfml-system.

If you use the dynamic versions of the SFML 1.6 libraries, you must also define the SFML_DYNAMIC symbol in the options for your project.
If you use the static version of the SFML 2.0 libraries, you must also define the SFML_STATIC symbol in the options for your project.
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

### <a name="tr-vs-arch"/>I keep getting ```fatal error LNK1112: module machine type 'XYZ' conflicts with target machine type 'ZYX'```. Help!

See [here](#tr-grl-64bit).

---

### <a name="tr-win"/>Windows

### <a name="tr-win-console"/>Why does a console attach itself to my project?

In Windows, if you compile your project, you will have a console that attaches itself behind your window. To avoid this, you must create the correct type of project or change its type after creation. Many IDEs have options allowing you to choose whether you want to create a console or a GUI application. It is however recommended to create an empty project and manually set its type afterwards. This ensures that nothing else is set automatically that might cause strange behaviour later on.

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

Though the library names might vary, especially regarding the version number.

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

SFML is under the [zlib/png license](http://www.opensource.org/licenses/zlib-license.php). You can use SFML for both open-source and proprietary projects, including paid or commercial ones. If you use SFML in your projects, a credit or mention is appreciated, but is not required.

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
* [TGUI](https://tgui.eu)
* [CEGUI](http://www.cegui.org.uk/wiki/index.php/Main_Page)
* [Guichan](https://code.google.com/p/guichan/)
* [libRocket](http://librocket.com/)

### <a name="libraries-gui-lib"/>Can you interface SFML with a GUI library?

Yes, you can! See examples for Qt, wxWidgets, and the native Win32 and X11 APIs in the [official tutorials](http://www.sfml-dev.org/tutorials/).

### <a name="libraries-video"/>Can I read video files with SFML?

SFML does not have a video playback module, but one can easily connect FFMPEG or similar libraries with SFML. There even exists a couple of maintained projects from some SFML users called [sfeMovie](http://lucas.soltic.etu.p.luminy.univ-amu.fr/sfeMovie/) and [Motion](http://en.sfml-dev.org/forums/index.php?topic=16221.0).

### <a name="libraries-thor"/>What exactly is Thor?

Thor is an open-source and cross-platform library written in the programming language C++. It is an extension to SFML and comes with high-level features that base on SFML and that are supposed to help in daily C++ routine, especially with respect to graphics and game programming.

The git repository is also hosted on [GitHub](https://github.com/Bromeon/Thor) and the official website can be found at: http://www.bromeon.ch/libraries/thor/

## <a name="misc"/>Miscellaneous

### <a name="misc-projects"/>Are there any famous projects with SFML?

As SFML is open source and has a permissive free software license, game creators are not forced to specify that they've used SFML in their games, thus it might well be that SFML has been used in bigger commercial projects, we don't know of. As for the known projects, there's a [dedicated page](Projects) on the SFML's wiki.

### <a name="misc-distr"/>How can I distribute my game?

### <a name="misc-upload"/>Where can I upload my game to?

There are many possible places to share your game and game related data with the world wide web and their users. [HashCookie](http://hashcookie.net/) is a simple and easy to use site. Another provider to consider is [MediaFire](http://www.mediafire.com/). Both are simple and fast file sharing hosts.

Also [SFML Projects](http://sfmlprojects.org/) is currently being developed as a place to put many of the projects made with SFML.

### <a name="misc-examples"/>Are there any example projects I can learn from?

Examples of the usage of each module are provided with SFML itself if you download the full SDK version. If you want full projects that make use of many features of SFML at the same time your best bet is to check out the [projects forum](http://en.sfml-dev.org/forums/index.php?board=10.0). People frequently showcase what they are working on there for the community to see. Often they might provide the source code as well, so if you feel brave enough you can look in there if you find a certain project interesting.