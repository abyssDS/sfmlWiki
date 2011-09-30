# Groogy's Development Environment
This is a open-source development environment to help out with developing rbSFML applications. It includes some debugging utilities and various features to make development easier. Feel free to add your own additions to the library. Thumb rule being that it is to help out with developing applications in Ruby and not take over it so it can't develop into framework, everything should be optional to use.

## Features
So far the library adds seven parts. A eight part is planned and on the way.

### Logger
The library comes with a logger pre-setup to STDOUT for any messages for the developer. The library itself uses this logger as well to output various failures.

### Graphical Messages
The library comes with a message class that let's you display messages graphically in it's own window.

### Code-On-The-Fly Loader
A thread run in the background monitoring the loaded files for changes. If a file is changed while the application is running then that file will be reloaded into memory updating the code currently running with the new changes. This gives us the possibility to have an application running while we are developing and see the changes tread into effect at the moment we save.

So far this handles any syntax errors that might arise but it is still in it's early age of development. For instance if we define a method in a module and then remove it this method will still exist in the ruby interpreter. 

### Console
A few console classes that acts pretty much like the IO operations of Kernel. The difference is that it is made to act like how you expect a console to do. Two classes was added that directly outputs the text into a graphical context like window or texture. 

### Resource Loader
A module that will load resources in another thread and which you can query for progress information. Pretty handy if you want to load things in the background and not get a huge pause from the load time. Can be used in conjunction with a loading screen or in-game to load the next level while playing. 

**NOTE!** Does not speed up loading in any way! More or less it should make it slower because of Ruby's GIL.

### Debugger Module
Should come up with a better name but for now it's called Debugger. What it lets us do is print common data to the
screen at real time showing us the performance of the application. So far it is capable of showing us the FPS and the memory usage by ruby.

### Garbage thread
Is now possible to create a thread that forces the garbage collector to be run from time to time. Because of the GIL we will loose a tiny bit of performance when using this but if you are creating a lot of temporary objects this is more or less necessary to keep a steady memory usage.

### Watches (Concept/Theory/Under development)
Haven't started on this yet but will be based on the console. Will allow the developer to specify the environment to watch a specific variable for any changes and report back trough a console window. This will let the developer see various values in real-time without clogging up the main application window.

## License
GDE is completely free to use for open-source and closed-source projects. Like SFML, it is licensed under the very permissive zlib/libpng license.

## Download
You can download the environment from the github page located [here](https://github.com/Groogy/GDE). Feel free to participate in the development of the environment by submitting code or letting know what you would like to see.

## Examples
### Simple example
Here's a simple example to get started:

```ruby
# main.rb
load './src/gde/gde.rb'

GDE.init( "Example 1" )
GDE.load( "print.rb" )

for index in 0..500
  print_message()
  sleep( 1 )
end

GDE.exit()
```
```ruby
# print.rb
def print_message()
  puts "Hello world!"
end
```

If the print message method in _print.rb_ is changed while running then it will be updated by the environment and a new message will be shown instead.