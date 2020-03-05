Notice: Most FAQ entries have been moved to [the official SFML FAQ](http://www.sfml-dev.org/faq.php).

The remaining entries here on the unofficial FAQ are maintained by the community.

# Frequently asked questions (FAQ)

**[Libraries for SFML](#libraries)**
- [Does SFML have a GUI package?](#libraries-gui-package)
- [Can you interface SFML with a non-SFML GUI library?](#libraries-gui-lib)
- [Can I read video files with SFML?](#libraries-video)
- [What exactly is Thor?](#libraries-thor)

**[Miscellaneous](#misc)**
- [Are there any famous projects that use SFML?](#misc-projects)
- [Where can I share my SFML project(s)?](#misc-share)

---

## <a name="libraries"/>Libraries for SFML

### <a name="libraries-gui-package"/>Does SFML have a GUI package?

No, SFML does not have a GUI module integrated into it, nor does it have an official GUI package; however, [you can use nearly any OpenGL-based GUI library with SFML](#libraries-gui-lib). Here is a (incomplete) list of commonly used GUI libraries:

* [CEGUI](http://cegui.org.uk/content/getting-started)
* [dear imgui](https://github.com/ocornut/imgui) and it's SFML-intended counterpart [imgui-sfml](https://github.com/eliasdaler/imgui-sfml)
* [Guichan](https://code.google.com/p/guichan/)
* [libRocket](https://github.com/libRocket/libRocket)
* [Nuklear](https://github.com/Immediate-Mode-UI/Nuklear)
* [SFGUI](https://github.com/TankOs/SFGUI)
* [TGUI](https://tgui.eu)

Some of these are intended for SFML use and are seamlessly incorporated into existing SFML projects (namely [imgui-sfml](https://github.com/eliasdaler/imgui-sfml), [SFGUI](https://github.com/TankOs/SFGUI), and [TGUI](https://tgui.eu)). [The others can be interfaced as well](#libraries-gui-lib).

### <a name="libraries-gui-lib"/>Can you interface SFML with a non-SFML GUI library?

Yes, you can! See example interfaces for Qt, wxWidgets, and the native Win32 and X11 APIs in the [official tutorials](http://www.sfml-dev.org/tutorials/).

### <a name="libraries-video"/>Can I read video files with SFML?

SFML does not have a video playback module, but one can easily connect [FFmpeg](https://www.ffmpeg.org/) or similar libraries with SFML. There are even a couple of maintained projects from SFML users. One is [sfeMovie](http://sfemovie.yalir.org/) and another is [Motion](http://en.sfml-dev.org/forums/index.php?topic=16221.0).

### <a name="libraries-thor"/>What exactly is Thor?

Thor is an open-source and cross-platform library written in the programming language C++. It is an extension to SFML and comes with high-level features that base on SFML and that are intended to help in daily C++ routine, especially with respect to graphics and game programming.

The git repository for it is hosted on [GitHub](https://github.com/Bromeon/Thor) and the official website can be found [here](http://www.bromeon.ch/libraries/thor/).

## <a name="misc"/>Miscellaneous

### <a name="misc-projects"/>Are there any famous projects with made with SFML?

As SFML is open source and has a permissive free software license, game creators are not forced to specify that they've used SFML in their games, thus it might well be that SFML has been used in bigger commercial projects. We don't know. As for the known projects, there's a [dedicated page](Projects) on the SFML's wiki. A list of known projects can also be found on [SFML's Wikipedia page](http://en.wikipedia.org/wiki/Simple_and_Fast_Multimedia_Library#Video_game_examples_using_SFML). One other place is the [SFML Projects](#sfml-projects-org) website.

### <a name="misc-share"/>Where can I share my SFML project(s)?

<a name="sfml-projects-org"/>[SFML Projects](http://sfmlprojects.org/) is currently being developed as a place to put projects made with SFML.

Their project's intent can be found on their [About](https://sfmlprojects.org/about) page:
> While the SFML forum provides a section for [projects](https://en.sfml-dev.org/forums/index.php?board=10.0), it happens too often that nice projects get lost in the depth of that sub-forum or that the archive with the game gets deleted. With the SFML Projects website we want to provide a platform to host, share and archive projects that were made with or for [SFML](https://sfml-dev.org)
