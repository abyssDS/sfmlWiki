In this tutorial, we will go through the steps that are required to build your game with SFML for Steam on Linux. As the steps for Windows and Mac are well covered in online tutorials, we are not repeating them here.

We are going to use CMake to build our project, for further reference on how to setup your project using CMake, see [[Build your SFML project with CMake|Tutorial:-Build-your-SFML-project-with-CMake]].

# Prerequisites

* If you don't have a project using CMake yet, have a look at [[Build your SFML project with CMake|Tutorial:-Build-your-SFML-project-with-CMake]] and build your minimal project. This is of course optional, but it will simplify the linking process.
* Download the Steam SDK from your Steamworks page (you need  to have developer access: [Steamworks SDK](https://partner.steamgames.com/?goto=%2Fdownloads%2Fsteamworks_sdk.zip) and extract it to so somewhere like _${PROJECT_SOURCE_DIR}/ext/steam-sdk_.
* You should have Steam installed on your Linux computer.
* Download the Steam Runtime here: [Steam Runtime](https://github.com/ValveSoftware/steam-runtime)

# CMake configuration

The main changes that need to be done to your CMake file include linking the Steam library. 
**Important**: The Steam runtime only supports CMake version 2.8.7 (Check [this](https://github.com/ValveSoftware/steam-runtime/issues/25) for the progress there), you may have to downgrade your required CMake minimum version.

```cmake
cmake_minimum_required(VERSION 2.8.7)

# feature flag to include steam
option(STEAM "Include steamworks API?" OFF)

# this should already exist in a form in your project
file(GLOB_RECURSE Your_FILES
	"${PROJECT_SOURCE_DIR}/include/*.h"
	"${PROJECT_SOURCE_DIR}/src/*.cpp"
# this is new
    if (STEAM)
        "${PROJECT_SOURCE_DIR}/ext/steam-sdk/public/steam/*.h"
        "${PROJECT_SOURCE_DIR}/ext/steam-sdk/public/steam/*.cpp"
    endif()
)

# this or similar should already be part of your CMakeLists.txt
target_link_libraries(${EXECUTABLE_NAME} ${SFML_LIBRARIES} ${SFML_DEPENDENCIES})

# link the steam api here. This one is for linux, you could also add checks for other operating systems.
if (STEAM)
     target_link_libraries(${EXECUTABLE_NAME} ${PROJECT_SOURCE_DIR}/ext/steam-sdk/redistributable_bin/linux64/libsteam_api.so)
endif()
```

# Check your build

First, check that the build works in your normal environment. Then, you're ready to build it inside the sandbox the steam runtime provides.

# Install the runtime

* Install chroot
```bash
sudo apt-get install chroot
```
* Navigate to the folder you installed your runtime. 
* Install the runtime, we recommend to install both 32 and 64bit versions, which then result in the respective builds. You may reach more people with your game ;)
```bash
./setup_shroot.sh --amd64
./setup_shroot.sh --i386
```

# Switch to the runtime

* Navigate to your game's source folder
* Enter the steam runtime sandbox (for a 64bit build, use amd64 instead of i386)
```bash
schroot --chroot steamrt_scout_i386
```
* You're now inside the runtime! Great.

# Build your game inside the runtime

* Use the normal steps to build your game, e.g.
```bash
mkdir build
cd build
cmake ..
make
```
* Et voilà! Your game should have built and linked everything inside the sandbox. Steam will use the same runtime when  executing on a player's computer, which means that it will use exactly the same SFML version and other libraries you used.
* You cannot run this version directly, but after upload via the Steam pipe, you should be ready to go.

-- Ironbell & Achpile