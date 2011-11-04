CMake allows your project to be built in various environments, including command-line make, Code::Blocks project files, Eclipse project files, etc.  It's used in SFML 2.0.

In this tutorial we'll write a simple CMake configuration file with centralized version numbering, and see how to integrate SFML in it.

# The source files

First, create a `cmake_modules` directory and copy [FindSFML.cmake](https://github.com/LaurentGomila/SFML/blob/master/cmake/Modules/FindSFML.cmake) in it.  FindXXXX is automatically searched by CMake's `find_package` command.

Then create a main.cpp, for instance:
```c++
#include "config.h"
#include <iostream>
#include <SFML/Window.hpp>
using namespace std;

int main(int argc, char* argv[]) {
  cout << "Version " << myproject_VERSION_MAJOR << "." << myproject_VERSION_MINOR << endl;

  glClear(GL_COLOR_BUFFER_BIT);
  while (App.IsOpened()) {
    sf::Event Event;
    while (App.GetEvent(Event)) {
      if (Event.Type == sf::Event::Closed)
	App.Close();
    }
    App.Display();
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
cmake_minimum_required(VERSION 2.6)
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
find_package(SFML 1.6 REQUIRED system window graphics network audio)
target_link_libraries(${EXECUTABLE_NAME} ${SFML_LIBRARIES})


# OpenGL
find_package(OpenGL)
include_directories(${OPENGL_INCLUDE_DIR})
target_link_libraries(${EXECUTABLE_NAME} ${OPENGL_LIBRARIES})
target_link_libraries(${EXECUTABLE_NAME} m)  # if you use maths.h

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
find_package(SFML 1.6 REQUIRED system window graphics network audio)
target_link_libraries(${EXECUTABLE_NAME} ${SFML_LIBRARIES})
```

* First we specify where we stored the FindSFML.cmake module.
* Then we request CMake to look for it in the system, and search for the specified modules (here I specified them all).
* Last, we tell CMake to link our executable with the SFML libraries that it just found. 

In the sample main.cpp, we used OpenGL, so we can add:
```cmake
# OpenGL
find_package(OpenGL)
include_directories(${OPENGL_INCLUDE_DIR})
target_link_libraries(${EXECUTABLE_NAME} ${OPENGL_LIBRARIES})
target_link_libraries(${EXECUTABLE_NAME} m)  # if you use maths.h
```

# Compilation

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

* [Reference for v2.8](http://www.cmake.org/cmake/help/cmake-2-8-docs.html#command:target_link_libraries2-8-docs.html)
* [CMake Tutorial](http://www.cmake.org/cmake/help/cmake_tutorial.html)

-- [Beuc](http://www.beuc.net/)