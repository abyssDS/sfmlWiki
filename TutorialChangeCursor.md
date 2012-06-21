# How to change your cursor

The cursor is something every computer user is familiar with and in fact is constantly staring at, yet many people don't even realize anymore that it's there and changes its shape every so often. Next to the functionality like pointing and clicking the cursor can show many different states and indicate possible actions. For example you'll get a selection cursor when hovering over a text or a hand shaped cursor could indicate a link etc.

Since SFML is not a framework nor a GUI system providing a native function for changing the mouse cursor doesn't fit its purpose. Here's where this tutorial comes in.
If you're making an application or game, you might want to be able to display a different cursor. There are two ways to change your cursor:

1. You can hide the default cursor and draw a sprite where the cursor should be
2. You can ask your OS to do it for you. (Windows & Linux)

This tutorial will cover both methods.

### What do you need?
You'll need [SFML](http://www.sfml-dev.org/download.php#2.0-rc), an [editor](http://www.vim.org/) and a [compiler](http://gcc.gnu.org/) (obviously these links are only suggestions).
Since I won't go into details regarding C++ or SFML, the tutorial requires you to have some basic knowledge on both topics.

## Hide and Draw
This task is fairly simple, it consists out of three required and one optional tasks:

1. Use `sf::RenderWindow::setMouseCursorVisible(bool)` to hide the cursor.
2. Set the position of the sprite to the position of the mouse.
3. (optional) Adjust the view to get the correct render position.
4. Draw the sprite to the screen.

Since it's so simple there's not more to talk about and an example shows a possible implementation:
```cpp
#include <SFML/Graphics.hpp>

int main()
{
    sf::RenderWindow window(sf::VideoMode(800, 600), "Hidden Cursor");
    window.setMouseCursorVisible(false); // Hide cursor
    
    sf::View fixed = window.getView(); // Create a fixed view
    
    // Load image and create sprite
    sf::Texture texture;
    texture.loadFromFile("cursor.png");
    sf::Sprite sprite(texture);
    
    while(window.isOpen())
    {
        sf::Event event;
        while(window.pollEvent(event))
        {
            if(event.type == sf::Event::Closed)
            {
                window.close();
            }
        }
        
        sprite.setPosition(static_cast<sf::Vector2f>(sf::Mouse::getPosition(window))); // Set position
        
        window.clear();
        window.setView(fixed);
        window.draw(sprite);
        window.display();
    }

    return EXIT_SUCCESS;
}
```

***

## Ask your Operation System (OS)

What are the advantages of using this method over the other?

* The OS will not duplicate the cursor in memory, because it is a shared resource.
* The OS will use a cursor that the user is used to. For example on windows 7 the user is not used to having an hour glass, but is familiar with some sort of an animated circle.

### Class Prototype

Since we have to store the cursor differently for the Linux as for the Windows platform, we've introduced an preprocessor switch for the include files and the private class members.

```cpp
#ifndef STANDARDCURSOR_HPP
#define STANDARDCURSOR_HPP

#include <SFML/System.hpp>
#include <SFML/Window.hpp>

#ifdef SFML_SYSTEM_WINDOWS
    #include <windows.h>
#elif defined(SFML_SYSTEM_LINUX)
    #include <X11/cursorfont.h>
    #include <X11/Xlib.h>
#else
    #error This OS is not yet supported by the cursor library.
#endif

namespace sf
{
    class StandardCursor
    {
    private:
		#ifdef SFML_SYSTEM_WINDOWS
            
		HCURSOR Cursor; /*Type of the Cursor with Windows*/
			
		#else

        XID Cursor;
        Display* display;

		#endif
    public:
        enum TYPE{ WAIT, TEXT, NORMAL, HAND /*,...*/ };
        StandardCursor(const TYPE t);
        void set(const sf::WindowHandle& aWindowHandle) const;
        ~StandardCursor();
    };
}

#endif // STANDARDCURSOR_HPP
```

### Class Implementation

Instead of 'breaking' the scope with preprocessor statements we'll define every function twice with a switch for each platform.

```cpp
#include "StandardCursor.hpp"

#ifdef SFML_SYSTEM_WINDOWS

sf::StandardCursor::StandardCursor(const sf::StandardCursor::TYPE t)
{
    switch(t)
    {
        case sf::StandardCursor::WAIT :
            Cursor = LoadCursor(NULL, IDC_WAIT);
        break;
        case sf::StandardCursor::HAND :
            Cursor = LoadCursor(NULL, IDC_HAND);
        break;
        case sf::StandardCursor::NORMAL :
            Cursor = LoadCursor(NULL, IDC_ARROW);
        break;
        case sf::StandardCursor::TEXT :
            Cursor = LoadCursor(NULL, IDC_IBEAM);
        break;
        //For more cursor options on Windows go here: http://msdn.microsoft.com/en-us/library/ms648391%28v=vs.85%29.aspx
    }
}

void sf::StandardCursor::set(const sf::WindowHandle& aWindowHandle) const
{
	SetClassLongPtr(aWindowHandle, GCLP_HCURSOR, reinterpret_cast<LONG_PTR>(Cursor));
}

sf::StandardCursor::~StandardCursor()
{
	// Nothing to do for destructor : no memory has been allocated (shared ressource principle)
}

#elif defined(SFML_SYSTEM_LINUX)

sf::StandardCursor::StandardCursor(const sf::StandardCursor::TYPE t)
{
    display = XOpenDisplay(NULL);
    
    switch(t)
    {
        case sf::StandardCursor::WAIT:
            Cursor = XCreateFontCursor(display, XC_watch);
        break;
        case sf::StandardCursor::HAND:
            Cursor = XCreateFontCursor(display, XC_hand1);
        break;
        case sf::StandardCursor::NORMAL:
            Cursor = XCreateFontCursor(display, XC_left_ptr);
        break;
        case sf::StandardCursor::TEXT:
            Cursor = XCreateFontCursor(display, XC_xterm);
        break;
        // For more cursor options on Linux go here: http://www.tronche.com/gui/x/xlib/appendix/b/
    }
}

void sf::StandardCursor::set(const sf::WindowHandle& aWindowHandle) const
{
    XDefineCursor(display, aWindowHandle, Cursor);
    XFlush(display);
}

sf::StandardCursor::~StandardCursor()
{
    XFreeCursor(display, Cursor);
    delete display;
    display = NULL;
}

#else
    #error This OS is not yet supported by the cursor library.
#endif
```

### Cursor Demonstration 

This section presents a fully functional demonstration of both cursor changing possibilities.

To get an handle to the window we will use SFML's `sf::Window::getSystemHandle()` function and then we will load the cursor with the OS specific implementation.

```cpp
#include <iostream>
#include <SFML/Graphics.hpp>
#include "StandardCursor.hpp"

int main()
{
	int choice = 0;
	while(choice != 1 && choice != 2)
	{
		std::cout << "\t1. Hide the cursor and draw your own." << std::endl;
		std::cout << "\t2. Let the OS handle the cursor." << std::endl;
		std::cout << "Choose your cursor behaviour: ";
		std::cin >> choice;
	}

    sf::RenderWindow window(sf::VideoMode(800, 600), "Cursor Demonstration");

	if(choice == 2)
    {
		sf::StandardCursor Cursor(sf::StandardCursor::HAND);
		Cursor.set(window.getSystemHandle());
	}
	else
		window.setMouseCursorVisible(false);
	
	sf::View fixed = window.getView();
	sf::Texture texture;
	if(!texture.loadFromFile("cursor.png"))
		return EXIT_FAILURE;
	sf::Sprite sprite(texture);
    
    while(window.isOpen())
    {
		sf::Event event;
		while(window.pollEvent(event))
			if(event.type == sf::Event::Closed)
				window.close();

		window.clear();

		if(choice == 1)
		{
			sprite.setPosition(static_cast<sf::Vector2f>(sf::Mouse::getPosition(window)));
			window.setView(fixed);
			window.draw(sprite);
		}

		window.display();
    }
    return EXIT_SUCCESS;
}
```

A *.zip file containing both, the code for `StandardCursor` class and the demonstration and an cursor image, can be obtained through this link: [cursor.zip](http://dev.my-gate.net/wp-content/uploads/2012/06/cursor.zip) (2.9 KB)

This wiki entry is partially based on the [old tutorial](http://www.sfml-dev.org/wiki/en/tutoriels/cursor), but has been renewed to also work with Windows 64bit and expanded with the example for the hidden cursor technique by eXpl0it3r.