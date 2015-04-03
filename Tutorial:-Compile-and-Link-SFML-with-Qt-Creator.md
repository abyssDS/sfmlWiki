This tutorial will first show you, how to build SFML with Qt creator and after that how to link your application against SFML.

# Building SFML
This guide will first cover compiling for Windows and then compiling for Linux, Ubuntu-esque.

## Windows
### Step 1
Visit https://github.com/LaurentGomila/SFML

And press "Download this repository as a zip file" button.
![ ](http://i.imgur.com/3IemL.png)

### Step 2
Unzip the downloaded file.

### Step 3
Download and Install the appropriate version of CMake for your operating system (e.g. win32).
Visit: http://www.cmake.org/cmake/resources/software.html

![ ](http://i.imgur.com/cNNPo.png)

### Step 4
Open Qt Creator (tested on Qt Creator version provided with the Qt SDK)

Go to: Tools->Options->Build & Run->CMake

Search for the CMake executable NOT the GUI executable but rather the command line one (e.g. cmake.exe)

### Step 5
Click the "Open Project..." button and look for the CMakelists.txt file in the SFML source folder that you have just downloaded.

### Step 6
Click on the Projects icon on the left hand side of the IDE. Choose Build Settings from the top of the menu. 
Expand the Build Environment roll-out and look for the PATH variable. Click Edit and insert the following text making sure not to delete anything that is already there.
`C:\QtSDK\mingw\bin;C:\qtsdk\qt\bin;`

### Step 7
Click on the Projects icon on the left hand side of the IDE.
Choose Build Settings from the top of the menu.
Click the Run CMake button from the Reconfigure Project option.

### Step 8
On the CMake wizard choose the compiler you would normally use with your Qt Creator projects.
And click the Run CMake button.
When done click finish.

### Step 9
Click on the hammer icon on the bottom left of the editor to build the project. 
 
_This process will build SFML - RELEASE VERSION on the directory specified._

### Step 10
In order to build the debug version of the libraries:
Click on the Projects icon on the left hand side of the IDE.
Choose Build Settings from the top of the menu.
Click the Run CMake button from the Reconfigure Project option. (Similar to step 8)
But pass as an argument

-DCMAKE_BUILD_TYPE=Debug

Click Finish
And rebuild the project.

# Linking SFML
Because the default compiler produces .a files instead of .lib we have to manually edit the project in 
order to include the libraries.

After creating a new project in Qt edit the .pro file in the and paste the following lines:

    LIBS += -LC:/SFML/qtcreator-build/lib

    CONFIG(release, debug|release): LIBS += -lsfml-audio -lsfml-graphics -lsfml-main -lsfml-network -lsfml-window -lsfml-system
    CONFIG(debug, debug|release): LIBS += -lsfml-audio-d -lsfml-graphics-d -lsfml-main-d -lsfml-network-d -lsfml-window-d -lsfml-system-d

The above system path applies to a windows OS and you have to change the `C:\SFML\qtcreator-build\lib\` with the path where you created the SFML libraries.
Mac users can follow a similar process which I have not tested but should work the same way. 
The last two lines you should include in the .pro file are:

    INCLUDEPATH += C:/SFML/include
    DEPENDPATH += C:/SFML/include

Which are self explanatory...

Gotchas:
* If you want your executables in the same folder as your project you have to turn off __Shadow Build__ from the build options.
* If the executable runs but does not show anything it's not an error...Copy the SFML dlls in the same directory as the executable __plus `libgcc_s_dw2-1.dll` and `mingwm10.dll`__ which are located in the `C:\QtSDK\mingw\bin` folder.

## Linux
### Step 1
Visit https://qt-project.org/downloads and download the latest version of Qt Creator for your linux installation, 32-bit or 64-bit. 
You may have to run the following commands to install Qt Creator.

~~~
chmod u+x qt_creator_download.file
sudo ./qt_creator_download.file
~~~

### Step 2
Visit https://github.com/LaurentGomila/SFML

And press "Download this repository as a zip file" button.
![ ](http://i.imgur.com/3IemL.png)

Unzip the downloaded file.

### Step 3
On an Ubuntu-esque system you need to prepare your system by running the following commands:

~~~
sudo apt-get update
sudo apt-get install build-essential g++ cmake libjpeg-dev libfreetype6-dev libGLEW-dev libxrandr-dev libopenal-dev libsndfile-dev libgl1-mesa-dev
~~~

This is what I had to install on my system.  The build process may have additional requirements depending on your system.

### Step 4
Open Qt Creator (tested on Qt Creator version 2.7.2 with Qt 5.1.0 64-Bit)

Go to: `Tools->Options->Build & Run->CMake`

In a terminal run: `which cmake`.  Copy the path to CMake into Qt Creator.

### Step 5
Click the "Open Project..." button and look for the CMakelists.txt file in the SFML source folder that you downloaded.

### Step 6
In the CMake Wizard I used the default build directory which was "SFML-build". Click "Next."

If you want to build the debug version of the libraries, in "Arguments:" put:
~~~
-DCMAKE_BUILD_TYPE=Debug
~~~

Click "Run CMake" and check for any missing dependencies. Qt Creator will let you know.
If you don't get any errors click "Finish."

### Step 7
To remove the Debug option and build a release version.
Click on the Projects icon on the left hand side of the IDE.
Choose Build Settings from the top of the menu.
Click the "Run CMake" button from the Reconfigure Project option.
Clear the "Arguments:" text box.

### Step 9
Click on the hammer icon on the bottom left of the editor to build the project. 

I then moved the lib folder from the build folder into the original SFML source folder.
You need the following folders.
~~~
SFML
|- include
|- lib
~~~

# Linking SFML in Linux
After creating a new Non-Qt Plain C++ project in Qt edit the .pro file in the and paste the following lines, with _/home/user/Projects/SFML_ being the path to SFML on your system.

~~~
LIBS += -L"/home/user/Projects/SFML/lib"

CONFIG(release, debug|release): LIBS += -lsfml-audio -lsfml-graphics -lsfml-network -lsfml-window -lsfml-system
CONFIG(debug, debug|release): LIBS += -lsfml-audio-d -lsfml-graphics-d -lsfml-network-d -lsfml-window-d -lsfml-system-d

INCLUDEPATH += "/home/user/Projects/SFML/include"
DEPENDPATH += "/home/user/Projects/SFML/include"
~~~

After completing the above procedure you should have successfully created an SFML application up and running in the Qt Creator IDE using its native compilers!