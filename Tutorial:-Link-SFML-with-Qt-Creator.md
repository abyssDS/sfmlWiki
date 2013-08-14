# Link SFML with Qt Creator 2.7+ and Qt5
#### Step 1
Download the latest stable version of SFML from http://www.sfml-dev.org/download.php

## Windows
Unzip into a folder called SFML and place the folder in a location where you can find it. Such as:

~~~
C:/SFML
~~~

Create a Qt Project and edit the _(project name)_.pro file to include the following:

~~~
LIBS += -LC:/SFML/lib

CONFIG(release, debug|release): LIBS += -lsfml-audio -lsfml-graphics -lsfml-main -lsfml-network -lsfml-window -lsfml-system
CONFIG(debug, debug|release): LIBS += -lsfml-audio-d -lsfml-graphics-d -lsfml-main-d -lsfml-network-d -lsfml-window-d -lsfml-system-d

INCLUDEPATH += C:/SFML/include
DEPENDPATH += C:/SFML/include
~~~

## Mac OSX
The Mac OSX download includes a install.sh script which will copy the required files to their respective locations.

Create the Qt Project and edit the _(project name)_.pro file to include the following:

~~~
LIBS += -L"/usr/local/lib"

CONFIG(release, debug|release): LIBS += -lsfml-audio -lsfml-graphics -lsfml-system -lsfml-network -lsfml-window
CONFIG(debug, debug|release): LIBS += -lsfml-audio -lsfml-graphics -lsfml-system -lsfml-network -lsfml-window

INCLUDEPATH += "/usr/local/include"
DEPENDPATH += "/usr/local/include"
~~~

## Linux
I have not tested on linux, but it should be very similar to Mac OSX.  Download SFML for linux and copy the files to /usr/local.  From inside the include and lib folder run the following commands:
~~~
SFML
|
|-include
|-- cp -R * /usr/local/include
|
|-lib
|-- cp -R * /usr/local/lib
~~~

You may need to adjust the file permissions.  The .pro file should be the same as OSX.

## C++11
If you are using a C++11 version of SFML then you need to add the following to your .pro file:

~~~
CONFIG += c++11
~~~

If you are having problems compiling try, Build->Clean All or Build->Clean Project and then try to build or run again.
