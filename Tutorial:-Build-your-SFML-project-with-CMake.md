CMake allows your project to be built in various environments, including command-line make, Code::Blocks project files, Eclipse project files, etc.  It's used in SFML 2.0.

In this tutorial we'll write a simple CMake configuration file with centralized version numbering, and see how to integrate SFML in it.  This example is not for compiling SFML using CMake but for creating an example project that utilizes an existing build of SFML.

# The source files

First, create a `cmake_modules` directory and copy [FindSFML.cmake](https://github.com/LaurentGomila/SFML/blob/master/cmake/Modules/FindSFML.cmake) in it.  FindXXXX is automatically searched by CMake's `find_package` command.

Then create a main.cpp, for instance:
```c++
#include "config.h"
#include <iostream>
#include <SFML/Graphics.hpp>
using namespace std;

int main(int argc, char* argv[]) {

  /* Code adapted from the SFML 2 "Window" example */

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

And place your project license (such as the [GNU GPL](http://www.gnu.org/copyleft/gpl.html)) in a file named `COPYING`.

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
find_package(SFML 2 REQUIRED system window graphics network audio)
target_link_libraries(${EXECUTABLE_NAME} ${SFML_LIBRARIES})
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

# Compilation

If you installed SFML to a nonstandard place and CMake cannot find it, you'll need to add the location of the libraries manually.  To do this, add an entry called `CMAKE_PREFIX_PATH`.  These are locations that CMake searches in addition to the usual directories.  Specify the type as `PATH` and for the value, put the path to the `lib` directory of SFML.  This will then pull the necessary libraries in.

In addition, CMake may complain that it cannot find the include files for the `SFML_INCLUDE_DIR` entry.  You will have to set this manually to the `include` directory.

To build with Make and the GCC compiler on the command-line:
```bash
cd build/  # a separate directory so we can easily remove all the CMake work files
cmake ..  # generate Makefile's
make
# or, if you want a more traditional output with the commands run:
make VERBOSE=1
```

To create Code::Blocks project files:
```bash
cmake .. -G "CodeBlocks - Unix Makefiles"
```
and open `project.cbp` with Code::Blocks.

# Installation

CMake supports the classic:
```bash
make install
```

It also supports DESTDIR, useful in packaging:
```bash
make install DESTDIR=`pwd`/myworkdir
```

You can also define `CMAKE_INSTALL_PREFIX` when invoking `cmake` to change the default install path (`/usr/local` under Unix).  DESTDIR would still override this setting.
```bash
cmake -D CMAKE_INSTALL_PREFIX=/usr ..
```

If you want to check a variable's value, open `CMakeCache.txt`, they are all there!

# Packaging

You then can use `cpack` to make a project release:
```bash
# Binary package:
cpack -C CPackConfig.cmake
# Source package:
cpack -C CPackSourceConfig.cmake
```
You will get a nice `myproject-1.0.tar.gz`, among others.

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


-- [Beuc](http://www.beuc.net/)