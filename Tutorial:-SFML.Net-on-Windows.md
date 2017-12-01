It's not really a mystery on how to build SFML.Net, but it may not be as intuitive and having multiple steps doesn't make it easy to remember them all. This tutorial should document some of the steps.

## Dependencies

SFML.Net builds on top of CSFML which  builds on top of SFML. This means we first have to build SFML, then CSFML and only then can we build SFML.Net.
Alternatively you can also get pre-compiled binaries for SFML and CSFML.

* Build SFML statically with statically linked runtime libraries.
* Build CSFML with statically linked SFML and runtime libraries.
* Open the provided Visual Studio solution for SFML.Net.
* Update the solution to your Visual Studio version.
* Make sure OpenTK is available if you intend to build the window or opengl example. Easiest way to install OpenTK is to download the nuget package for the two projects in question.
* Build the SFML.Net solution or just the needed projects.
* Copy the created SFML.Net DLLs and the CSFML DLLs next to your binary and run it.