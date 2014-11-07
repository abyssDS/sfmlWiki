# Introduction

Why build from source?  SFML is under constant revision and improvement.  Bugs in the stable download are found and fixed relatively quickly.  If you encounter a bug in the stable package, chances are it could already be fixed.  

## What if I discovered a bug?

When a bug is discovered it is best to check the [forums](http://en.sfml-dev.org/forums/) to either confirm the bug or ask whether what you are seeing could be a bug.  If the condition is determined to be a bug, then it's time to create a new [issue](https://github.com/LaurentGomila/SFML/issues).  Issues should only be created for confirmed bugs.  If you suspect a bug, please use the forums first.

# Getting Started

This tutorial starts from a clean install of Ubuntu 14.x, other versions may work, but are untested.
Ensure the latest graphic card drives are installed.

## Setup

Before we can compile SFML we need to install the necessary dependencies.  To do this, we will use the Terminal.
With an open terminal, run the following commands: 

    sudo apt-get update
    sudo apt-get install build-essential
    sudo apt-get install git
    
This is the minimum required to build SFML.  We still need to get the dependencies, but from here we are going to learn the hard way first.

## Learning the hard way

The terminal window should have opened to your home folder and end in `:~$`
If not you can get back to your home folder with this command, `cd ~`

From the home folder we will create a place for the SFML source.  

    mkdir Projects
    cd Projects
    
Now its time to get the SFML source.

    git clone https://github.com/LaurentGomila/SFML.git SFML_SRC

The last part of that command tells git to place the source in a folder called `SFML_SRC`. If you leave off this part of the command, the folder will be called just `SFML`.

Now change into the `SFML_SRC` directory and list the contents.

    cd SFML_SRC
    ls 

Inside this directory you should see a file named `CMakeLists.txt`





