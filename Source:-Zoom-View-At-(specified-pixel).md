# ZoomViewAt free function

Free function that zooms a view at a specified pixel rather than the centre of the window/view.

Please note that C++11 is required for this exact code but it should be easy enough to adapt if you are still unwilling to upgrade ;)

## Function
```c++
void zoomViewAt(sf::Vector2i pixel, sf::RenderWindow& window, float zoom)
{
	const sf::Vector2f beforeCoord{ window.mapPixelToCoords(pixel) };
	sf::View view{ window.getView() };
	view.zoom(zoom);
	window.setView(view);
	const sf::Vector2f afterCoord{ window.mapPixelToCoords(pixel) };
	const sf::Vector2f offsetCoords{ beforeCoord - afterCoord };
	view.move(offsetCoords);
	window.setView(view);
}
```

Please note that the pixel is that actual pixel location of the window and not the co-ordinate of that pixel.

## Usage
To zoom at the mouse position when the mouse wheel is scrolled (placed inside the .pollEvent(event) loop):

```c++
if (event.type == sf::Event::MouseWheelScrolled)
{
	if (event.mouseWheelScroll.delta > 0)
		zoomViewAt({ event.mouseWheelScroll.x, event.mouseWheelScroll.y }, window, (1.f / zoomAmount));
	else if (event.mouseWheelScroll.delta < 0)
		zoomViewAt({ event.mouseWheelScroll.x, event.mouseWheelScroll.y }, window, zoomAmount);
}
```

where event is of type sf::Event,  
and zoomAmount is of type float.  
If zoomAmount is 1.1f, it will zoom in or out by 10%;  
if zoomAmount is 1.3f, it will zoom in or out by 30%.

## Example
Here's a full example, which also include the function:

```c++
#include <SFML/Graphics.hpp>
#include <iostream>

void zoomViewAt(sf::Vector2i pixel, sf::RenderWindow& window, float zoom)
{
	const sf::Vector2f beforeCoord{ window.mapPixelToCoords(pixel) };
	sf::View view{ window.getView() };
	view.zoom(zoom);
	window.setView(view);
	const sf::Vector2f afterCoord{ window.mapPixelToCoords(pixel) };
	const sf::Vector2f offsetCoords{ beforeCoord - afterCoord };
	view.move(offsetCoords);
	window.setView(view);
}

int main()
{
	sf::RenderWindow window(sf::VideoMode(800, 600), "\"Zoom View At\" example", sf::Style::Default);
	window.setFramerateLimit(60);
	sf::View view(window.getDefaultView());
	const float zoomAmount{ 1.1f }; // zoom by 10%

	sf::Texture texture;
	if (!texture.loadFromFile("ag80QzQ.png")) // feel free to download this image from here: http://i.imgur.com/ag80QzQ.png or just use your own
	{
		std::cerr << "Could not load image.";
		return EXIT_FAILURE;
	}
	sf::Sprite sprite(texture);
	sprite.setScale({ 0.51f, 0.51f }); // fit default image into default window size. you may need to adjust this if you use a different image

	while (window.isOpen())
	{
		sf::Event event;
		while (window.pollEvent(event))
		{
			if (event.type == sf::Event::Closed || event.type == sf::Event::KeyPressed && event.key.code == sf::Keyboard::Escape)
				window.close();
			else if (event.type == sf::Event::KeyPressed)
			{
				if (event.key.code == sf::Keyboard::BackSpace)
				{
					view = window.getDefaultView();
					window.setView(view);
				}
			}
			else if (event.type == sf::Event::MouseWheelScrolled)
			{
				if (event.mouseWheelScroll.delta > 0)
					zoomViewAt({ event.mouseWheelScroll.x, event.mouseWheelScroll.y }, window, (1.f / zoomAmount));
				else if (event.mouseWheelScroll.delta < 0)
					zoomViewAt({ event.mouseWheelScroll.x, event.mouseWheelScroll.y }, window, zoomAmount);
			}
		}

		window.clear();
		window.draw(sprite);
		window.display();
	}
	
	return EXIT_SUCCESS;
}
```

**Written by Hapax ([Github](http://github.com/hapaxia) | [SFML forum](http://en.sfml-dev.org/forums/index.php?action=profile;u=13086))**