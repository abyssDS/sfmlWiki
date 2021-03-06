# Introduction
Building SFML for iOS is mostly the same as [building with cmake](https://www.sfml-dev.org/tutorials/latest/compile-with-cmake.php) for any other platform, just with the iOS specific toolchain

## Requirements
* Xcode (with command line tools)
* CMake
* SFML 2.5.0 or later

### Dependencies
All iOS dependencies are provided with the SFML source code, so you should not need anything else


# Configuring your build
To build for iOS, you need to set `CMAKE_TOOLCHAIN_FILE` to the one provided with the SFML source code at `./cmake/toolchains/iOS.toolchain.cmake`. You'll also need to make sure to use the Xcode generator:

## Configuring with the command line
```
cd SFML  
mkdir build && cd build  
cmake .. -GXcode -DCMAKE_TOOLCHAIN_FILE=../cmake/toolchains/iOS.toolchain.cmake
```

## Configuring with CMake GUI
  * Click the "Configure" button.
  * In the pop-up window, choose "Xcode" as the generator for this project. Select "Specify toolchain file for cross-compiling". Press Continue.
  * Set the "Specify the Toolchain file" line to `[YOUR_SFML_SOURCE_PATH]/cmake/toolchains/iOS.toolchain.cmake`

Finally, press "Generate". This will create an Xcode project called "SFML.xcodeproj" inside your chosen build directory.

## Compiling SFML
Like any other platform you can build with the `cmake --build` command, or if you open the xcode project you can build normally from there


# Xcode project

## Creating an iOS target in Xcode
**Note: As mentioned previously, we will be using Xcode project for Mac created using SFML's Xcode template. [See this tutorial for installation.](https://www.sfml-dev.org/tutorials/2.4/start-osx.php) This will let us create a cross-platform (Mac and iOS) SFML program.**

### Create a new Xcode project.
* Open Xcode, and select `File > New > Project...`
* In the window that pops up, select the "SFML App" template from the "macOS" tab.  
![Xcode 1](http://i.imgur.com/YLaQtyI.png)
* Fill in the required options, select C++11 if you wish to use it, and create your project. Choose the folder where your project will be created.

You should now have a working Xcode project which compiles for Mac. Test that it does by using the arrow in the top left corner. If it doesn't, refer to the SFML tutorial for Xcode.

### Create a target for iOS
* In Xcode, select `File > New > Target...` to add a new target for iOS.
* In the window that pops up, select the "Game" template from the "iOS" tab.  
![Xcode 2](http://i.imgur.com/JyGJx7d.png)
  * **We will start from this template and modify it heavily so that it works with SFML. This is currently necessary as SFML does not provide default iOS templates like it does for macOS.**
* Select "Objective-C" as the language, "Universal" for the devices, and uncheck the inclusion of unit tests or ui tests. Make sure you're adding the target to your current project.  
![Xcode 3](http://i.imgur.com/Lw8LSfr.png)

Your project should now look like this:  
![Xcode 4](http://i.imgur.com/0fdMkVE.png)
**Notes:**
* The "Tutorial" target will compile for Mac, and the "iOS" target for iOS. If you want to rename the "Tutorial" target to "macOS" for clarity, you can do so by double left-clicking slowly on its name and entering a new name. For consistency, you'll probably want to edit the name of the scheme for Mac too. To keep this simple, we'll be keeping things as they are for this tutorial.
* If you want to change the iOS target's compiled application name, you can change its "Display Name" value in the "General tab" shown in the screenshot.

## Setting up the iOS target
**Tip:** To access the project settings and target settings, you can select the blue Xcode project icon file in the top left of the project navigator. Clicking on it opens up the view in the screenshot above, from which you can click on the targets to edit their settings.  

### Cleaning up unneeded files
* In the project navigator, delete all the files from the **"iOS"** folder except for "Assets.xcassets" and "Info.plist". When prompted, select "Move to Trash" for the "iOS" folder files, but "Remove References" for the "Frameworks" folder files.  
![Xcode 5](http://i.imgur.com/gzMy4Gk.png)

### Setting the project's settings so that it doesn't conflict with the iOS target
* Go back to the project settings view (see tip), and **select the project**. Open the "Build Settings" tab.
  * **Make sure to select the "All" and "Levels" views**. When adding the following values, we will be editing the second column, with the name of the project as its title.
  * Under "Architectures", select the "Supported Platforms" line (see image for reference)  
![Xcode 6](http://i.imgur.com/LTUdNna.png)
    * Press delete to delete its value.
    * Also delete the "Architectures" line with the same technique.
  * Under "Search Paths"
    * Delete the "Framework Search Paths" line

### Setting the macOS target's settings
* In the project settings view (see tip), **select the "[project name]" target**. This is your "macOS" target if you have renamed it. Open the "Build Settings" tab.
  * **Make sure to select the "All" and "Levels" views**.
  * Under "Search Paths"
    * Add "one line to "Framework Search Paths": `/Library/Frameworks/`. **Make sure you are adding it to the target, and not to the whole project.**
  * Under "User-Defined"
    * For each line starting with "SFML_***", select the value defined in the **project** (third column), **cut and paste** it to the macOS **target** (second column).
    * Once finished, it should look like this:  
![Xcode 7](http://i.imgur.com/GsdYIg3.png)

### Setting the iOS target's settings
* In the project settings view (see tip), **select the "iOS" target**. Open the "Build Phases" tab.
  * In "Compile Sources", add all the *.cpp files from the main project folder which you wish to compile for iOS. You can also do this by opening a *.cpp and setting its "Target Membership" values in the right sidebar.
    * Note: Do not try to add "ResourcePath.mm" to the iOS target as it will not compile for iOS.
  * In "Link Binary With Libraries", drag-and-drop all the libs you created earlier from `SFML/build/ios-install/lib/` (**except for libfreetype.a and libjpeg.a, as they are included in the extlibs**).
  * Also in "Link Binary With Libraries", drag-and-drop all the libs from `SFML/extlibs/libs-ios/`
  * Finally, click on the little "+" icon below "Link Binary With Libraries", and add the following iOS frameworks:
    * "CoreMotion.framework"
    * "OpenAL.framework"
    * "Foundation.framework"
    * "QuartzCore.framework"
    * "OpenGLES.framework"
    * "UIKit.framework"
    * **For SFML 2.5.0 and later:**
      * "libz.tbd"
      * "libbz2.tbd"
* Open the "Build Settings" tab for the iOS target.
  * **Make sure to select the "All" and "Levels" views**. When adding the following values, we will be editing the second column, titled "iOS".
  * Under "Architectures", set "Architectures" to "Other..." and edit the value to
    * **For SFML 2.4.2 and earlier:** `$(ARCHS_STANDARD_32_BIT)`
    * **For SFML 2.5.0 and later:** `x86_64` and `arm64`
  * Under "Search Paths"
    * If you see `/Library/Frameworks/` in "Framework Search Paths", delete that line.
    * Add one line to "Header Search Paths": `/Users/[username]/Documents/Libraries/SFML/include/`
    * Add two lines to "Library Search Paths": `/Users/[username]/Documents/Libraries/SFML/build/ios-install/lib/` and `/Users/[username]/Documents/Libraries/SFML/extlibs/libs-ios/`
    * Note: You can also simply drag-and-drop the folder into the pop-up window and Xcode will add a relative path for you.
    * Note 2: Make sure to escape any spaces with a backslash (`\`) or by including the entire path in quotations marks (but not both)
  * Under "User-Defined"
    * Delete the value for each line starting with "SFML_***" if it is in the "iOS" target (second column).
  * Open the "General" tab for the iOS target.
    * Under "Deployment Info", delete the value `Main` from the "Main Interface" line and leave it empty.

## Enabling shared resources between targets
To share resources and have them properly found by the application during runtime, we need to export the resources outside of the main project folder, so that once the app is compiled, they end up in the appropriate "Resources" folder inside of the application.
* Quit Xcode.
* Open Finder, and navigate to where the Xcode project is located. It should look something like this:  
![Finder 1](http://i.imgur.com/xKXYJk6.png)
* Create a new folder called "Assets"
  * You could name this folder whatever you want, because we won't even see the name of this folder inside. However, **do not** name it "resources" or "Resources", or the name of any folder within it. If you do so, Xcode will fail to compile or to open your application in the simulator.
  * Drag-and-drop all the resources (images, sounds and fonts) from the main project folder (in this case, "Tutorial") into the newly created "Assets" folder.
* It should now look like this:  
![Finder 2](http://i.imgur.com/ofvH5sm.png)
  * Note: For a more clear organization format, you **could** move all source and header files into their own folder, instead of having them reside in the "[project name]" folder which contains the Info.plist for the macOS target. You could then have a "macOS" folder with the Info.plist for Mac, similar to the "iOS" folder, and a folder containing all files shared between the two targets. You would have to recreate the groups in Xcode after doing so for this to work properly.
* Reopen Xcode, and delete the "Resources" folder from the project navigator. At the pop-up, choose "Remove References".
* Right-click on the **project file**, and select "New Group".
  * Name this group "Resources"
  * Open a Finder window, open the "Assets" folder, and drag-and-drop its **contents** (not the folder itself) into the "Resources" group in the Xcode project navigator.
  * At the pop-up, select "Create folder references" and select both targets.
* Your Xcode project navigator should now look like this:  
![Xcode 8](http://i.imgur.com/mpE1DqQ.png)

## Editing the last files
You're almost there!  
* In main.cpp
  * Change `int main(int, char const**)` to `int main()` (Otherwise, this causes the application to hang on startup)
  * Add the following line to the top of the file:  
```
#ifdef __APPLE__  
    #include "TargetConditionals.h"  
    #if TARGET_OS_IPHONE  
        #include <SFML/Main.hpp>  
    #endif  
#endif  
```
* In ResourcePath.hpp, replace this line `std::string resourcePath(void);` with this:  
```
#include <SFML/Config.hpp>

#ifdef SFML_SYSTEM_MACOS
    std::string ResourcePath(void);
#else
    inline std::string ResourcePath()
    {
        return "";
    }
#endif
```
* Edit the "Info.plist" in the "iOS" folder
  * Due to a bug with Xcode (8.3.3) it seems that for the project to compile successfully you need to set "Bundle Name" to `$(TARGET_NAME)` instead of `$(PRODUCT_NAME)`

# Compile
In the top-left corner, choose the "iOS" target from the drop-down. Click on the arrow to compile for the simulator.