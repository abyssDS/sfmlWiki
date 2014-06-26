## Introduction
Crosscompiling SFML for Android can be a bit tricky at times, depending on your host system. Unfortunately it's not as straightforward as compiling SFML for the actual host machine. This guide is meant to help you getting started. It won't go too far into detail. You'll sill have to attain at least some knowledge on your own on how to properly utilize the Android SDK/NDK.

## Requirements
Before you can start, you'll need the proper development environment. Basically, you'll need the following things installed (some might be installed already, e.g. to build SFML for your host machine):

* CMake
* Git
* Android SDK
* Android NDK

## Setup
Before you can start, there are a few things to note, that will make the following steps easier.
* Update your `PATH` environment variable to include the following paths:
  * *[Path to CMake]*/bin
  * *[Path to Git]*/bin
  * *[Path to SDK]*/tools
  * *[Path to SDK]*/platform-tools
  * *[Path to NDK]*

  Make sure to use backslashes (`\`) for these paths on Windows.
* Set a new environment variable `ANDROID_NDK` pointing to the directory where you've put the Android NDK. Make sure to use forward slashes (`/`) even on Windows, like `C:/Android/NDK`.

## Retrieving and Building SFML
Follow these steps to download, build and install SFML:
* Open a console or terminal window and go to a directory where you'd like to place your SFML sources and build files, e.g. `/home/code`.
* Clone the official SFML repository:

        git clone https://github.com/LaurentGomila/SFML.git SFML

  This will create a sub directory `SFML` containing all build files.
* Enter the new sub directory:

        cd SFML

* Create a build directory and enter it:

        mkdir build && cd build

* You can repeat the following steps for all available architectures. Unfortunately, you can't build all targets for SFML at once. The following lines create a `armeabi` build. If you'd like to build for any other target, just replace all occurences. Other valid targets would be `armeabi-v7a`, `mips`, and `x86`.
  * Create a sub directory and enter it:

        mkdir armeabi && cd armeabi

  * Now invoke CMake. Make sure to pass all parameters:

            cmake -DANDROID_ABI=armeabi -DCMAKE_TOOLCHAIN_FILE=../../cmake/toolchains/android.toolchain.cmake ../..
  * Wait for the process to complete. There might be a few warnigns regarding the toolchain(s), but you shouldn't see any other warnings or error messages.
  * This will create a makefile or project for you, based on your current host system.
  * Use it to build and then install SFML. For example, under Linux you'd issue the following commands:

        make && sudo make install
  * If everything went fine, this should have copied the created binaries as well as header files and dependencies to your Android NDK's `source` directory.
  * You're now ready to use SFML in the NDK together with devices understanding the compiled target files (in this example `armeabi`).

## Building and Executing the Android Example
*Not written yet.*