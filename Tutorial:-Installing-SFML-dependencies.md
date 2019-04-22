In order to build SFML on Linux, several libraries and their development headers need to be installed first. There is an exhaustive list available at the [SFML website](https://www.sfml-dev.org/tutorials/2.4/compile-with-cmake.php#installing-dependencies).

But you are using Ubuntu, this simple script should install all the dependencies through Ubuntu's APT (Advanced Packaging Tool) library.
```bash
#!/usr/bin/env bash
apt-get install         \
    libfreetype6-dev    \
    libjpeg-dev         \
    libx11-dev          \
    libxrandr-dev       \
    libx11-xcb-dev      \
    libxcb-randr0-dev   \
    libxcb-image0-dev   \
    libxcb1-dev         \
    libgl1-mesa-dev     \
    libudev-dev         \
    libopenal-dev       \
    libflac-dev         \
    libvorbis-dev -y
```