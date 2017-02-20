## Introduction
This guide provides comprehensive steps to install & configure the environment to cross-compile SFML for Android. If you have any issues, please post in this page's thread located at the SFML wiki forums.

## Download Dependencies
Before you can start, you'll need to download the following software.

* MinGW-64 - [here](https://sourceforge.net/projects/mingw-w64/files/latest/download)
* Android Studio - [here](https://developer.android.com/studio/index.html#win-bundle)
* Android NDK **v12b** - [here](https://developer.android.com/ndk/downloads/older_releases.html)
* Git **Portable** - [here](https://git-scm.com/download/win)
* CMake Installer - [here](http://ant.apache.org/bindownload.cgi)
* Apache Ant - [here](http://ant.apache.org/bindownload.cgi)
* JDK Installer - [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

## Install Dependencies
* Install MinGW-64 to "c:\projects\temp", move and rename "c:\projects\temp\mingw32" as "c:\mingw".
* Install Android Studio to "c:\projects\Android Studio" & "c:\projects\sdk".
* Extract Android NDK into "c:\projects\", rename directory to "ndk".
* Install Git Portable to "c:\projects\git".
* Install CMake to "c:\projects\cmake".
* Extract Apache Ant into "c:\projects\", rename directory to "apache-ant".
* Install Java JDK.
* Open "c:\projects\sdk\SDK Manager.exe", tick "Android 5.0.1 (API 21)", remember the API version you select.

## Configure Environment Variables
* We need to add the paths of these tools to the environment path variable, so CMD may find the tools we call upon.
* Open CMD as **administrator** from the start menu, paste the following command:
```
setx /M PATH "%PATH%;C:\projects\cmake\bin;C:\projects\sdk\tools;C:\projects\sdk\platform-tools;C:\projects\ndk;C:\projects\apache-ant\bin;C:\mingw\bin" && setx /M ANDROID_NDK "C:/projects/ndk"
```

## Compile SFML
* Open CMD.
* Run the following commands:
```
cd c:\projects
git\bin\git clone https://github.com/SFML/SFML.git SFML
cd SFML && mkdir build && cd build
mkdir armeabi-v7a && cd armeabi-v7a
set PATH=%PATH:c:\projects\git\bin;=%
cmake -G "MinGW Makefiles" -DANDROID_ABI=armeabi-v7a -DCMAKE_TOOLCHAIN_FILE=../../cmake/toolchains/android.toolchain.cmake ../..
mingw32-make install
```

## Compile Android Example
* Plug your phone into your computer, **enable USB debugging**. Google how to if you do not know.
* Make sure you answer any prompts at any stage of running these next commands.
* Run the following commands:
```
cd c:\projects\SFML\examples\android && rmdir bin /s /q
android update project --target "android-21" --path .
ndk-build
ant debug && adb install bin/NativeActivity-debug.apk
adb shell am start -a android.intent.action.MAIN -n com.example.sfml/android.app.NativeActivity
```

## Bonus - Pong for Android
* I've tweaked the bundled Pong game to work slightly better on Android.
* Replace the contents of "c:\projects\SFML\examples\android\jni\main.cpp" with the following: http://pastebin.com/wPuANqax
* Repeat the commands in the "Compile Android Example" section.