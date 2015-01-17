### Most FAQ entries have been moved to [the official SFML FAQ](http://www.sfml-dev.org/faq.php).

# Frequently asked questions (FAQ)

**[Building and Using SFML](#build-use)**
- [I want to fuse the libraries into one. (Not recommended)](#build-fuse)

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