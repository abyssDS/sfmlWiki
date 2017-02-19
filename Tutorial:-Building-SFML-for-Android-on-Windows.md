## Introduction
This guide provides comprehensive steps to install & configure the environment to cross-compile SFML for Android. If you have any issues, please post in this page's thread located at the SFML wiki forums.

## Download Dependencies
Before you can start, you'll need to download the following software.

* MinGW-64 - [here](https://developer.android.com/studio/index.html#downloads)
* Android Studio - [here](https://developer.android.com/studio/index.html#win-bundle)
* Android NDK **v12b** - [here](https://developer.android.com/ndk/downloads/older_releases.html)
* Git **Portable** - [here](https://git-scm.com/download/win)
* CMake Installer - [here](http://ant.apache.org/bindownload.cgi)
* Apache Ant - [here](http://ant.apache.org/bindownload.cgi)
* JDK Installer - [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

## Install Dependencies
* Install MinGW-64 to "c:\android\temp", move and rename "c:\android\temp\mingw32" to "c:\mingw".
* Install Android Studio to "c:\android\Android Studio" & "c:\android\sdk".
* Extract Android NDK into "c:\android\", rename directory to "ndk".
* Install Git Portable to "c:\android\git".
* Install CMake to "c:\android\cmake".
* Extract Apache Ant into "c:\android\", rename directory to "apache-ant".
* Install Java JDK.

## Configure Environment
* Open "c:\android\sdk\SDK Manager.exe", tick "Android 5.0.1 (API 21)", remember the API version you select.
* Open CMD as administrator from the start menu, paste the following command:
```
setx /M PATH "%PATH%;c:\android\cmake\bin;c:\android\git\bin;c:\android\sdk\tools;c:\android\sdk\platform-tools;c:\android\ndk;c:\android\apache-ant\bin;C:\mingw\bin" && setx /M ANDROID_NDK "c:/android/ndk"
```

## Compile SFML
* Open CMD.
* Run the following commands:
```
cd c:\android
git clone https://github.com/SFML/SFML.git SFML
cd SFML && mkdir build && cd build
mkdir armeabi-v7a && cd armeabi-v7a
set PATH=%PATH:c:\android\git\bin;=%
cmake -G "MinGW Makefiles" -DANDROID_ABI=armeabi-v7a -DCMAKE_TOOLCHAIN_FILE=../../cmake/toolchains/android.toolchain.cmake ../..
mingw32-make install
```

## Compile Android Example
* Plug your phone into your computer, enable USB debugging. Google how to if you do not know.
* Make sure you answer any prompts at any stage of running these next commands.
* Run the following commands:
```
cd c:\android\SFML\examples\android && rmdir bin /s /q
android update project --target "android-21" --path .
ndk-build
ant debug install && adb install bin/NativeActivity-debug.apk
adb shell am start -a android.intent.action.MAIN -n com.example.sfml/android.app.NativeActivity
```

## Bonus - Pong for Android
* I've tweaked the bundled Pong game to work slightly better on Android.
* Replace the contents of "c:\android\SFML\examples\android\jni\main.cpp" with the following: http://pastebin.com/wPuANqax
* Repeat the commands in the "Compile Android Example" section.