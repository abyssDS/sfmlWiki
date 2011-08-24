# Groogy's Development Environment
This is a open-source development environment to help out with developing rbSFML applications. It includes some debugging utilities and various features to make development easier. Feel free to add your own additions to the library. Thumb rule being that it is to help out with developing applications in Ruby and not take over it so it can't develop into framework, everything should be optional to use.

## Features
So far the library adds three parts.

### Logger
The library comes with a logger pre-setup to STDOUT for any messages for the developer. The library itself uses this logger as well to output various failures.

### Graphical Messages
The library comes with a message class that let's you display messages graphically in it's own window.

### Code-On-The-Fly Loader
A thread run in the background monitoring the loaded files for changes. If a file is changed while the application is running then that file will be reloaded into memory updating the code currently running with the new changes. This gives us the possibility to have an application running while we are developing and see the changes tread into effect at the moment we save.

So far this handles any syntax errors that might arise but it is still in it's early age of development. For instance if we define a method in a module and then remove it this method will still exist in the ruby interpreter. 

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