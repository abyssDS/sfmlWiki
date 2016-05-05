In my opinion the easiest way to build SFML is to use the "MinGW Makefiles" CMake generator and the CMake GUI. So here is a brief tutorial on how to build SFML that way.

### Prepare your Development Environment

If you do not have your MinGW compiler's `bin` directory in `PATH`, you can temporarily add it by opening a command prompt (`Windows Key` + `R` and enter `cmd` + `Enter`):

`set PATH=<path_to_your_mingw_compiler>\bin;%PATH%`

Next you can start the CMake GUI from the command prompt by typing `cmake-gui`. If CMake's `bin` directory hasn't been added to `PATH`, I highly recommend you do so in a similar fashion as mentioned above.

![Set PATH and open the CMake GUI](http://i.imgur.com/PVw3aP3.png)

Then setup the path to SFML and a build path for, I usually just use the subdirectory `build\`.

![Set the source & build directory](http://i.imgur.com/Yi9MXRA.png)

### Setup CMake

Then press the `Configure` button and pick the `MinGW Makefiles` generator.

**Note:** If you use MSYS you need to use the `MSYS Makefiles` generator instead.

![Pick the generator](http://i.imgur.com/PkIAR8f.png)

As a next step you can change the various CMake variables. A full explanations of these options can be found in the [official SFML CMake tutorial](http://www.sfml-dev.org/tutorials/2.2/compile-with-cmake.php). One of the important parts is to change the `CMAKE_INSTALL_PREFIX`, since you'll get permission issues with the default CMake value and you essentially really don't want your library files installed there. Also don't pick the build directory, otherwise the install will end up messy. I personally prefer to use the sub-directory `/install`.

![Install path option](http://i.imgur.com/Of7hAnd.png)

After you've changed everything you wanted to change you can click on `Generate` and you should get the two done-messages.

![Successful configuration](http://i.imgur.com/W0Smlp0.png)

### Actually Building SFML

CMake merely generates makefiles and project files, as such SFML as a C++ library hasn't been built yet and thus we need to this next. For that we switch back to the command prompt and change the directory to the build directory we've previously set in CMake.

Since we want everything directly installed to the previously specified `CMAKE_PREFIX_INSTALL` directory, we'll now run the makefile's `install` target. Additionally since we're most likely using a multi-core processor, we really want to get things building quickly and thus make use of the `-jN` flag, where `N` is an integer that specifies how many threads shall be used. Keep in mind that this can really freeze your system if you pick a number too high for your system. Enter the command `mingw32-make install -jN` as shown below.

**Note:** If you're using MSYS (or nuwen's MinGW compiler) you need to use `make` instead of `mingw32-make`.

![Build the install target](http://i.imgur.com/3K2FODc.png)

Wait for the process to finish and check the `install` directory.

:tada: **Congratulations you have successfully built SFML!** :tada:

----------

### Using Code::Blocks Instead

If you really want a Code::Blocks project file, then you can repeat the first few steps from above, but instead of the `MinGW Makefiles` CMake generator pick the `CodeBlocks - MinGW Makefiles`.

![Code::Blocks generators](http://i.imgur.com/6yvTsT7.png)

Adjust the settings as mentioned above, including `CMAKE_INSTALL_PREFIX` and press the `Generate` button. After everything has been generated, you can go and open up the Code::Blocks project file in the `build` directory.

![Code::Blocks project file](http://i.imgur.com/mcu0Bfj.png)

In Code::Blocks you can then easily pick your preferred target (`install` is recommended) and click on the build button.

![Code::Blocks target](http://i.imgur.com/m8HptW8.png)

Wait for the process to finish and check the `install` directory.

:tada: **Congratulations you have successfully built SFML!** :tada: