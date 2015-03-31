In order to build SFML on Linux, several libraries need to be installed first.

Here is a simple shell script for Ubuntu that uses apt-get. Even if you use a different package manager, the package names may still be useful for you.
```sh
#!/bin/sh

# sfml-window
apt-get install libx11-dev -y
apt-get install libxcb1-dev -y
apt-get install libx11-xcb-dev -y
apt-get install libxcb-randr0-dev -y
apt-get install libxcb-image0-dev -y
apt-get install libgl1-mesa-dev -y
apt-get install libudev-dev -y

# sfml-graphics
apt-get install libfreetype6-dev -y
apt-get install libjpeg-dev -y

# sfml-audio
apt-get install libopenal-dev -y
apt-get install libflac-dev -y
apt-get install libvorbis-dev -y
```