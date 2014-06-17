# Manage different screens in a game

When speaking about screens, I mean Menu screen, Config screen, Game screen, etc... Those screens you find in every games. The problem here is that each screen can be compared to a small SFML application : Each screen will have its own events and will use some variables useless for other screens.

So, we need to separate each screen in order to avoid conflicts. With SFML, it's very simple to do that! You just have to create a cScreen class which will represent each screen. This is an virtual object and it's containing only one method:

```cpp
class cScreen
{
public :
    virtual int Run (sf::RenderWindow &App) = 0;
};
```

This object is made very simple in order to allow great compatibility with every application structure. Run is a method called from the main function, and it returns a positive integer if an other screen must be shown. Otherwise, it returns -1 and the application exits. With a simple object, we have now some screens containing their owns variables and an SFML event loop...

The old SFML Event loop is replaced with :

```cpp
while (screen >= 0)
{
screen = Screens[screen]->Run(App);
}
```

Therefore, you just have to specify the first screen and get into the loop. Each screen will returns, saying the app continues with an other screen or stops. Each Run method get the sf::RenderWindow variable in order to get events and draw sprites, etc...

More, screens are objects created in the main function of the application. So, they keep their values as long as they are alive. It permits to get back to Config menu while being in game and return to the game without being forced to save everything.

Here is a demo app using the cScreen object :

\- A simple main function

```cpp
#include <iostream>
#include <SFML/Graphics.hpp>
#include "screens.hpp"

int main(int argc, char** argv)
{
	//Applications variables
	std::vector<cScreen*> Screens;
	int screen = 0;

	//Window creation
	sf::RenderWindow App(sf::VideoMode(640, 480, 32), "SFML Demo 3");

	//Mouse cursor no more visible
	App.setMouseCursorVisible(false);

	//Screens preparations
	screen_0 s0;
	Screens.push_back(&s0);
	screen_1 s1;
	Screens.push_back(&s1);

	//Main loop
	while (screen >= 0)
	{
		screen = Screens[screen]->Run(App);
	}

	return EXIT_SUCCESS;
}
```


\- screens.hpp contains the cScreen include, plus the hpp of each screens of the game. Here, screen_0 is a menu, and screen_1 is the game. Each screen inherits from cScreen :

```cpp
#ifndef SCREENS_HPP_INCLUDED
#define SCREENS_HPP_INCLUDED

//Basic Screen Class
#include "cScreen.hpp"

//Including each screen of application
#include "screen_0.hpp"
#include "screen_1.hpp"

#endif // SCREENS_HPP_INCLUDED
```


\- screen_0.hpp contains the object definition and its code. "Run" is like the main loop you can have in a simple SFML application. The constructor defines the first value of private members which are finally more like static variables in our case. This permits to have a fade presentation at first loading, since the menu will appear directly. Once background is shown, we use two string to make a menu in which you can navigate with direction's key. Choosing "Play" will go to the game, "Exit" will exit the application by returning -1 to the main function :

```cpp
#include <iostream>
#include "cScreen.hpp"

#include <SFML/Graphics.hpp>

class screen_0 : public cScreen
{
private:
	int alpha_max;
	int alpha_div;
	bool playing;
public:
	screen_0(void);
	virtual int Run(sf::RenderWindow &App);
};

screen_0::screen_0(void)
{
	alpha_max = 3 * 255;
	alpha_div = 3;
	playing = false;
}

int screen_0::Run(sf::RenderWindow &App)
{
	sf::Event Event;
	bool Running = true;
	sf::Texture Texture;
	sf::Sprite Sprite;
	int alpha = 0;
	sf::Font Font;
	sf::Text Menu1;
	sf::Text Menu2;
	sf::Text Menu3;
	int menu = 0;

	if (!Texture.loadFromFile("presentation.png"))
	{
		std::cerr << "Error loading presentation.gif" << std::endl;
		return (-1);
	}
	Sprite.setTexture(Texture);
	Sprite.setColor(sf::Color(255, 255, 255, alpha));
	if (!Font.loadFromFile("verdanab.ttf"))
	{
		std::cerr << "Error loading verdanab.ttf" << std::endl;
		return (-1);
	}
	Menu1.setFont(Font);
	Menu1.setCharacterSize(20);
	Menu1.setString("Play");
	Menu1.setPosition({ 280.f, 160.f });

	Menu2.setFont(Font);
	Menu2.setCharacterSize(20);
	Menu2.setString("Exit");
	Menu2.setPosition({ 280.f, 220.f });

	Menu3.setFont(Font);
	Menu3.setCharacterSize(20);
	Menu3.setString("Continue");
	Menu3.setPosition({ 280.f, 160.f });

	if (playing)
	{
		alpha = alpha_max;
	}

	while (Running)
	{
		//Verifying events
		while (App.pollEvent(Event))
		{
			// Window closed
			if (Event.type == sf::Event::Closed)
			{
				return (-1);
			}
			//Key pressed
			if (Event.type == sf::Event::KeyPressed)
			{
				switch (Event.key.code)
				{
				case sf::Keyboard::Up:
					menu = 0;
					break;
				case sf::Keyboard::Down:
					menu = 1;
					break;
				case sf::Keyboard::Return:
					if (menu == 0)
					{
						//Let's get play !
						playing = true;
						return (1);
					}
					else
					{
						//Let's get work...
						return (-1);
					}
					break;
				default:
					break;
				}
			}
		}
		//When getting at alpha_max, we stop modifying the sprite
		if (alpha<alpha_max)
		{
			alpha++;
		}
		Sprite.setColor(sf::Color(255, 255, 255, alpha / alpha_div));
		if (menu == 0)
		{
			Menu1.setColor(sf::Color(255, 0, 0, 255));
			Menu2.setColor(sf::Color(255, 255, 255, 255));
			Menu3.setColor(sf::Color(255, 0, 0, 255));
		}
		else
		{
			Menu1.setColor(sf::Color(255, 255, 255, 255));
			Menu2.setColor(sf::Color(255, 0, 0, 255));
			Menu3.setColor(sf::Color(255, 255, 255, 255));
		}

		//Clearing screen
		App.clear();
		//Drawing
		App.draw(Sprite);
		if (alpha == alpha_max)
		{
			if (playing)
			{
				App.draw(Menu3);
			}
			else
			{
				App.draw(Menu1);
			}
			App.draw(Menu2);
		}
		App.display();
	}

	//Never reaching this point normally, but just in case, exit the application
	return (-1);
}
```

\- screen_1.hpp contains the game itself. We can't exit from the game directly. If you push Escape, you'll go on the Main Menu. If you move the "player" and go to the menu, you can go back to the game and your position will stay the same. Because your object screen_1 has not been destroyed :

```cpp
#include <iostream>
#include "cScreen.hpp"

#include <SFML/Graphics.hpp>

class screen_1 : public cScreen
{
private:
	float movement_step;
	float posx;
	float posy;
	sf::RectangleShape Rectangle;
public:
	screen_1(void);
	virtual int Run(sf::RenderWindow &App);
};

screen_1::screen_1(void)
{
	movement_step = 5;
	posx = 320;
	posy = 240;
	//Setting sprite
	Rectangle.setFillColor(sf::Color(255, 255, 255, 150));
	Rectangle.setSize({ 10.f, 10.f });
}

int screen_1::Run(sf::RenderWindow &App)
{
	sf::Event Event;
	bool Running = true;

	while (Running)
	{
		//Verifying events
		while (App.pollEvent(Event))
		{
			// Window closed
			if (Event.type == sf::Event::Closed)
			{
				return (-1);
			}
			//Key pressed
			if (Event.type == sf::Event::KeyPressed)
			{
				switch (Event.key.code)
				{
				case sf::Keyboard::Escape:
					return (0);
					break;
				case sf::Keyboard::Up:
					posy -= movement_step;
					break;
				case sf::Keyboard::Down:
					posy += movement_step;
					break;
				case sf::Keyboard::Left:
					posx -= movement_step;
					break;
				case sf::Keyboard::Right:
					posx += movement_step;
					break;
				default:
					break;
				}
			}
		}

		//Updating
		if (posx>630)
			posx = 630;
		if (posx<0)
			posx = 0;
		if (posy>470)
			posy = 470;
		if (posy<0)
			posy = 0;
		Rectangle.setPosition({ posx, posy });

		//Clearing screen
		App.clear(sf::Color(0, 0, 0, 0));
		//Drawing
		App.draw(Rectangle);
		App.display();
	}

	//Never reaching this point normally, but just in case, exit the application
	return -1;
}
```

Here it is, while using this class, you've separated each part of your application not changing very much your app.

Good luck to you now!