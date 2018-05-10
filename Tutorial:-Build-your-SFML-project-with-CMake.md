CMake allows your project to be built in various environments, including most popular Makefiles and IDE's. It is available from SFML 2.0.

# !!! This tutorial doesn't work for SFML 2.5 anymore !!!

In this tutorial we'll write a simple CMake configuration file with centralized version numbering, and see how to integrate SFML in it.  This example is not for compiling SFML using CMake but for creating an example project that utilizes an existing build of SFML.

# The source files

First, create a `cmake_modules` directory and copy [FindSFML.cmake](https://github.com/SFML/SFML/blob/master/cmake/Modules/FindSFML.cmake) in it.  FindXXXX is automatically searched by CMake's `find_package` command.

Then create a main.cpp, for instance:
```c++
#include "config.h"
#include <iostream>
#include <SFML/Graphics.hpp>
using namespace std;

int main(int argc, char* argv[]) {

  // Code adapted from the SFML 2 "Window" example.

  cout << "Version " << myproject_VERSION_MAJOR << "." << myproject_VERSION_MINOR << endl;

  sf::Window App(sf::VideoMode(800, 600), "myproject");

  while (App.isOpen()) {
    sf::Event Event;
    while (App.pollEvent(Event)) {
      if (Event.type == sf::Event::Closed)
	App.close();
    }
    App.display();
  }
}
```

We use a `config.h` file that is built by CMake, let's also create a `config.h.in` file:
```c++
#define myproject_VERSION_MAJOR @myproject_VERSION_MAJOR@
#define myproject_VERSION_MINOR @myproject_VERSION_MINOR@
```

And place your project license (such as the [zlib license](https://www.zlib.net/zlib_license.html)) in a file named `COPYING`.

# CMake configuration

Let's describe our project configuration in the `CMakeLists.txt` file:
```cmake
#Change this if you need to target a specific CMake version
cmake_minimum_required(VERSION 2.6)


# Enable debug symbols by default
# must be done before project() statement
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
endif()
# (you can also set it on the command line: -D CMAKE_BUILD_TYPE=Release)

project(myproject)

# Set version information in a config.h file
set(myproject_VERSION_MAJOR 1)
set(myproject_VERSION_MINOR 0)
configure_file(
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
  )
include_directories("${PROJECT_BINARY_DIR}")

# Define sources and executable
set(EXECUTABLE_NAME "myproject")
add_executable(${EXECUTABLE_NAME} main.cpp)


# Detect and add SFML
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake_modules" ${CMAKE_MODULE_PATH})
#Find any version 2.X of SFML
#See the FindSFML.cmake file for additional details and instructions
find_package(SFML 2 REQUIRED network audio graphics window system)
if(SFML_FOUND)
  include_directories(${SFML_INCLUDE_DIR})
  target_link_libraries(${EXECUTABLE_NAME} ${SFML_LIBRARIES} ${SFML_DEPENDENCIES})
endif()


# Install target
install(TARGETS ${EXECUTABLE_NAME} DESTINATION bin)


# CPack packaging
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/COPYING")
set(CPACK_PACKAGE_VERSION_MAJOR "${myproject_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${myproject_VERSION_MINOR}")
include(CPack)
```

The interesting part is:
```cmake
# Detect and add SFML
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake_modules" ${CMAKE_MODULE_PATH})
find_package(SFML 2 REQUIRED COMPONENTS network audio graphics window system)
target_link_libraries(${EXECUTABLE_NAME} ${SFML_LIBRARIES} ${SFML_DEPENDENCIES})
```

* First we specify where we stored the FindSFML.cmake module.
* Then we request CMake to look for it in the system, and search for the specified modules (here I specified them all).
* Last, we tell CMake to link our executable with the SFML libraries that it just found. 

# Additional libraries

Here are a few example of commonly used C/C++ libraries:

```cmake
# OpenGL
find_package(OpenGL REQUIRED)
include_directories(${OPENGL_INCLUDE_DIR})
if (OPENGL_FOUND)
  target_link_libraries(${EXECUTABLE_NAME} ${OPENGL_LIBRARIES})
  # or: target_link_libraries(${EXECUTABLE_NAME} ${OPENGL_gl_LIBRARY})
  target_link_libraries(${EXECUTABLE_NAME} m)  # if you use maths.h
endif()
```
```cmake
# boost::filesystem
#set(Boost_ADDITIONAL_VERSIONS "1.78" "1.78.0" "1.79" "1.79.0")
find_package(Boost 1.34.0 REQUIRED system filesystem)
if(Boost_FOUND)
  include_directories(${Boost_INCLUDE_DIRS})
  target_link_libraries(${EXECUTABLE_NAME} ${Boost_LIBRARIES})
endif()
```
```cmake
# pkg-config-based library
include(FindPkgConfig)
pkg_check_modules(yaml-cpp REQUIRED yaml-cpp>=0.2.5)
if(yaml-cpp_FOUND)
  include_directories(${yaml-cpp_INCLUDE_DIRS})
  link_directories(${yaml-cpp_LIBRARY_DIRS})
endif()
add_executable(${EXECUTABLE_NAME} ${SOURCES})
target_link_libraries(${EXECUTABLE_NAME} ${yaml-cpp_LIBRARIES})
...
```
Note: because `link_directories` needs to be called before a target is created, if you use the pkg-config snippet, you need to move your `add_executable` and `target_link_libraries` calls _after_ the call to `link_directories`.

# Configuration

To configure a CMake project, execute `cmake ..` (where `..` is the relative path to your CMakeLists.txt created above).

At this point you may also specify a specific Generator by adding the `-G` flag to the cmake command, e.g. `cmake .. -G"Xcode"`. For a full list of generators run `cmake --help`

At this point your project may complain that SFML is not found. If you have installed SFML to a non-standard place, then you will need to set the SFML_ROOT variable so it can be found, e.g. `cmake .. -G"Xcode" -DSFML_ROOT=/path/to/sfml`
 
# Compilation

At this point you should have generated project files in the folder you executed cmake from. You can use these directly as you normally would, or you can use cmake's build command: `cmake --build .`
 
# Installation

To install your project, build the install target. If you're using the cmake build command, you can add `--target install`

You can also define `CMAKE_INSTALL_PREFIX` when invoking `cmake` to change the default install path (`/usr/local` under Unix).  
```bash
cmake -D CMAKE_INSTALL_PREFIX=/usr ..
```

If you want to check a variable's value, open `CMakeCache.txt`, they are all there!

# Packaging

To package your project into an archive/installer, just build the package target

```bash
cmake --build . --target package
```

By default this usually just creates a compressed archive with your installed files, but it is highly configurable. See CMake/CPack documentation for more info

# Further information

* CMake has built-in help, for instance:
```bash
$ cmake --help-variable CMAKE_SOURCE_DIR
cmake version 2.8.5
  CMAKE_SOURCE_DIR
       The path to the top level of the source tree.

       This is the full path to the top level of the current CMake source
       tree.  For an in-source build, this would be the same as
       CMAKE_BINARY_DIR.
```

* [CMake reference for version 2.8](http://www.cmake.org/cmake/help/cmake-2-8-docs.html)
* [CMake Tutorial](http://www.cmake.org/cmake/help/cmake_tutorial.html)
* [CMake tutorial en français](http://geenux.wordpress.com/2009/12/27/utilisation-de-cmake/)

-- [Beuc](http://www.beuc.net/) & the other members of the SFML community