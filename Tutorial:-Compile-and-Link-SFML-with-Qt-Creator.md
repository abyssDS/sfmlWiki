This tutorial will first show you, how to build SFML with Qt creator and after that how to link your application against SFML.

# Building SFML
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
    CONFIG(debug, debug|release): LIBS += -lsfml-audio-d -lsfml-graphics-d -lsfml-main-d -lsfml-network-d -lsfml-window-d lsfml-system-d

The above system path applies to a windows OS and you have to change the `C:\SFML\qtcreator-build\lib\` with the path where you created the SFML libraries.
Linux and Mac users can follow a similar process which I have not tested but should work the same way. 
The last two lines you should include in the .pro file are:

    INCLUDEPATH += C:/SFML/include
    DEPENDPATH += C:/SFML/include

Which are self explanatory...

Gotchas:
* If you want your executables in the same folder as your project you have to turn off __Shadow Build__ from the build options.
* If the executable runs but does not show anything it's not an error...Copy the SFML dlls in the same directory as the executable __plus `libgcc_s_dw2-1.dll` and `mingwm10.dll`__ which are located in the `C:\QtSDK\mingw\bin` folder.

After completing the above procedure you should have successfully created an SFML application up and running in the Qt Creator IDE using its native compilers!