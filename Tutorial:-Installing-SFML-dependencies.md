In order to build SFML on Linux, several libraries need to be installed first.

Here is a simple shell script for Ubuntu that uses apt-get. Even if you use a different package manager, the package names may still be useful for you.
```sh
#!/bin/sh

apt-get install libglew1.6-dev -y
apt-get install libx11-dev -y
apt-get install libxrandr-dev -y
apt-get install libxcb1-dev -y
apt-get install libx11-xcb-dev -y
apt-get install libxcb-icccm4-dev -y
apt-get install libxcb-randr0-dev -y
apt-get install libxcb-util0-dev -y
apt-get install libxcb-image0-dev -y
apt-get install libxcb-keysyms1-dev
apt-get install libxcb-ewmh1-dev
apt-get install libfreetype6-dev -y
apt-get install libsndfile-dev -y
apt-get install libjpeg-dev -y
apt-get install libopenal-dev -y
```