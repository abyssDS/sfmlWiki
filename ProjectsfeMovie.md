# sfeMovie project

sfeMovie is a small C++ library that allows you playing movies in SFML based applications. It relies on SFML for the rendering process and FFmpeg for the decoding process. It supports both audio and video, and basic controls. This project has been written to work closely with SFML and tries to keep the same easy-to-use paradigm,
while being coherent with SFML's conventions.

###Quick links###

1. [How to use sfeMovie?](#howto)
2. [License](#license)
3. [System requirements](#requirements)
4. [Dependencies](#dependencies)
5. [Download links](#downloads)
6. [Supported file formats](#formats)
7. [Current status about features](#status)
8. [Contact](#contact)
9. [Thanks](#thanks)


## <a name="howto" />How to use sfeMovie ? [ [Top] ](#top)

Here is a sample code showing how to use sfeMovie, the instructions for each OS follow.

```cpp
#include <SFML/Graphics.hpp>
#include <Movie.h>

int main()
{
    sf::RenderWindow window(sf::VideoMode(640, 480), "SFE Movie Player");
    sfe::Movie movie;

    if (!movie.OpenFromFile("movie.avi"))
        return 1;

    movie.ResizeToFrame(0, 0, 640, 480);
    movie.Play();

    while (window.IsOpened())
    {
        sf::Event ev;
        while (window.GetEvent(ev))
        {
            if (ev.Type == sf::Event::Closed)
                window.Close();
        }

        window.Clear();
        window.Draw(movie);
        window.Display();
    }

    return 0;
}
```


###Linux###
N/A

###Mac OS X###


**With Xcode**

- Add the directory where you put Movie.h to the headers search path.
- Drag every library from "lib" and "extlib" to your project.
- Add a Copy File build phase to your project, and set the "Destination" of this phase to "Frameworks".
- Add all of the imported libraries to this build phase.
- Write your code and build your app.

**Without Xcode**

- Write your code.
- Add the directory where you put Movie.h to the header search path (through option -I).
- Add the directory/ies containing the libraries originally found in "lib" and "extlib" to the library search path (through option -L)
- Compile your application.
- Copy the libraries to your_app.app/Contents/Frameworks

###Windows###
N/A

## <a name="license" />License [ [Top] ](#top)

libsfe-movie is statically linked against FFmpeg. Thus you don't need to care about
the FFmpeg libraries, but **libsfe-movie is itself covered by the LGPL license**.

Basically, this means you can use sfeMovie for ANY project without ANY restriction until
libsfe-movie is linked dynamically to your software. As soon as you link libsfe-movie statically,
your software is also covered by the LGPL licence and you MUST provide the sources of your
application/library. 

See <http://www.gnu.org/licenses/lgpl.html> for more information on LGPL.

## <a name="requirements" />System requirements [ [Top] ](#top)
## <a name="dependencies" />Dependencies [ [Top] ](#top)
## <a name="downloads" />Download links [ [Top] ](#top)
## <a name="formats" />Supported file formats [ [Top] ](#top)
## <a name="status" />Current status about features [ [Top] ](#top)
## <a name="contact" />Contact [ [Top] ](#top)
## <a name="thanks" />Thanks [ [Top] ](#top)
