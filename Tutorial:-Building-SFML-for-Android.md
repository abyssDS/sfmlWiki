## Introduction
Crosscompiling SFML for Android can be a bit tricky at times, depending on your host system. Unfortunately it's not as straightforward as compiling SFML for the actual host machine. This guide is meant to help you getting started. It won't go too far into detail. You'll still have to attain at least some knowledge on your own on how to properly utilize the Android SDK/NDK.

These steps are experimental, feel free to edit this page and make improvements.

## Requirements
Before you can start, you'll need the proper development environment. Basically, you'll need the following things installed (some might be installed already, e.g. to build SFML for your host machine):

* [CMake](https://cmake.org/download/)
* Git
* [Android SDK](https://developer.android.com/studio/index.html#downloads)
* [Android NDK](https://developer.android.com/ndk/downloads/index.html)
* [Gradle](https://gradle.org)

## Setup
Before you can start, there are a few things to note, that will make the following steps easier.
* Update your `PATH` environment variable to include the following paths:
  * *[Path to CMake]*/bin
  * *[Path to Git]*/bin
  * *[Path to SDK]*/tools
    * Note: On Mac, this can be found at /Users/[Username]/Library/Android/sdk/tools
  * *[Path to SDK]*/platform-tools
    * Note: On Mac, this can be found at /Users/[Username]/Library/Android/sdk/platform-tools
  * *[Path to NDK]*

  Make sure to use backslashes (`\`) for these paths on Windows.  
  On Mac, use the command `sudo nano /etc/paths` to edit `PATH`. Make sure to escape any spaces with a backslash (`\`).  
  Note: You may encounter further problems if your directory paths contain spaces. It is recommended you avoid spaces in any of your directories.  

## Retrieving and Building SFML
Follow these steps to download, build and install SFML:
* Open a console or terminal window and go to a directory where you'd like to place your SFML sources and build files, e.g. `/home/code`.
* Clone the official SFML repository:

        git clone https://github.com/SFML/SFML.git SFML

  This will create a sub directory `SFML` containing all build files.
* Enter the new sub directory:

        cd SFML

* Create a build directory and enter it:

        mkdir build && cd build

* You can repeat the following steps for all available architectures. Unfortunately, you can't build all targets for SFML at once. The following lines create a `armeabi-v7a` build. If you'd like to build for any other target, just replace all occurences. Other valid targets would be `mips`, and `x86`.
  * Create a sub directory and enter it:

          mkdir armeabi-v7a && cd armeabi-v7a

  * Now invoke CMake. Make sure to pass all parameters:

          cmake -DCMAKE_SYSTEM_NAME=Android -DCMAKE_ANDROID_NDK=/path/to/ndk -DCMAKE_ANDROID_ARCH_ABI=armeabi-v7a -DCMAKE_ANDROID_STL_TYPE=c++_static -DCMAKE_BUILD_TYPE=Debug ../..

  * If you've got multiple toolsets installed, like Visual Studio and MinGW, you might want to pick the type of project or makefile to create. You can do this by adding a parameter like `-G "MinGW Makefiles"` (note the quotes).
  * If building process fails due to error 'No toolchain for ABI 'armeabi' found in the NDK_PATH' then you need to update your CMake.
  * Important: It can be tricky to get this process to work with Visual Studio! I'd recommend you use MinGW's make (which is essentially GNU make). See the previous step to create the proper makefiles.
  * Wait for the process to complete. There might be a few warnings regarding the toolchain(s), but you shouldn't see any other warnings or error messages.
  * This will create a makefile or project for you, based on your current host system.
  * Use it to build and then install SFML. For example, under Linux you'd issue the following commands:

          make && sudo make install

  * If everything went fine, this should have copied the created binaries as well as header files and dependencies to your Android NDK's `source` directory.
  * You're now ready to use SFML in the NDK together with devices understanding the compiled target files (in this example `armeabi-v7a`).

## Building and Executing the Android Example
Go into the example directory:

        cd SFML/examples/android

Add a "local.properties" file with the following contents, specifying the android SDK and NDK paths:

        sdk.dir=/path/to/android-sdk
        ndk.dir=/path/to/android-ndk

Now you should be able to build project with gradle:

        gradle build

If this results in an error stating "Could not open terminal for stdout: could not get termcap entry" then set the `TERM` variable to `dumb` (e.g. run `TERM=dumb gradle build` on linux).

If all goes well then you can now install the apk to a device or emulator by running

        gradle installDebug

## Troubleshooting

### Windows 10 and Android build

If you're running Windows 10, don't use the Power Shell, use the regular command line (cmd) to run the cmake and make commands.