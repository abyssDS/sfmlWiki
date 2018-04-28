# RotateViewAt free function

Free function that rotates a view around a specified co-ordinate rather than the centre of the view.

Since this function requires sine and cosine, you should remember to include the cmath header where you include the function:  
`#include <cmath>`

Please note that a minimum of C++11 is required for this exact code but it should be easy enough to adapt if you are still unwilling to upgrade ;)

## Function
```c++
void rotateViewAt(sf::Vector2f coord, sf::View& view, float rotation)
{
	const sf::Vector2f offset{ coord - view.getCenter() };
	const float rotationInRadians{ rotation * 3.141592653f / 180.f };
	const float sine{ std::sin(rotationInRadians) }; // remember to "#include <cmath>" for this
	const float cosine{ std::cos(rotationInRadians) }; // remember to "#include <cmath>" for this
	const sf::Vector2f rotatedOffset{ cosine * offset.x - sine * offset.y, sine * offset.x + cosine * offset.y };
	view.rotate(rotation);
	view.move(offset - rotatedOffset);
}
```

Please note that the co-ordinate is the co-ordinate within the view's co-ordinate system. To use an absolute pixel, you can simply map the pixel to a co-ordinate before the rotation.

## Usage
To rotate at the mouse position when the mouse wheel is scrolled (placed inside the .pollEvent(event) loop):

```c++
if (event.type == sf::Event::MouseWheelScrolled)
{
	sf::Vector2f mouseCoord{ window.mapPixelToCoords({ event.mouseWheelScroll.x, event.mouseWheelScroll.y }) };
	if (event.mouseWheelScroll.delta > 0)
		rotateViewAt(mouseCoord, view, rotateAmount));
	else if (event.mouseWheelScroll.delta < 0)
		rotateViewAt(mouseCoord, view, -rotateAmount);
}
```

where event is of type sf::Event,  
and rotateAmount is of type float, representing the desired rotation in degrees.  
If rotateAmount is positive, the rotation of the view will be clockwise; note this means that everything in the view will appear to rotate anti-clockwise/counter-clockwise.

## Example
Here's a full example, which also includes the function:

```c++
#include <SFML/Graphics.hpp>
#include <iostream>
#include <cmath> // sin, cos

void rotateViewAt(sf::Vector2f coord, sf::View& view, float rotation)
{
	const sf::Vector2f offset{ coord - view.getCenter() };
	const float rotationInRadians{ rotation * 3.141592653f / 180.f };
	const float sine{ std::sin(rotationInRadians) }; // remember to #include <cmath> for this
	const float cosine{ std::cos(rotationInRadians) }; // remember to #include <cmath> for this
	const sf::Vector2f rotatedOffset{ cosine * offset.x - sine * offset.y, sine * offset.x + cosine * offset.y };
	view.rotate(rotation);
	view.move(offset - rotatedOffset);
}

int main()
{
	// amount to rotate in degrees
	constexpr float rotationAmount{ 5.f };

	// creates a view that ranges from -1 to 1 in both vertical and horizontal directions
	sf::View view;
	view.setSize({ 2.f, 2.f });
	view.setCenter({ 0.f, 0.f });

	// prepares a rectangle that has its origin set to its centre. its position is automatically (0, 0), which means its centre is at the centre of the view (and therefore window)
	sf::RectangleShape rectangle({ 1.f, 0.7f });
	rectangle.setOrigin(rectangle.getSize() / 2.f);

	// create simple window
	sf::RenderWindow window(sf::VideoMode(800, 600), "Rotate View Around Point (instead of centre)", sf::Style::Default);
	window.setFramerateLimit(60);

	// main loop
	while (window.isOpen())
	{
		// normal event loop
		sf::Event event;
		while (window.pollEvent(event))
		{
			switch (event.type)
			{
			case sf::Event::Closed:
				window.close();
				break;
			case sf::Event::MouseWheelScrolled:
			{
				sf::Vector2f mouseCoord{ window.mapPixelToCoords({ event.mouseWheelScroll.x, event.mouseWheelScroll.y }, view) }; // convert mouse's pixel position to view's co-ordinate system
				if (event.mouseWheelScroll.delta > 0)
					rotateViewAt(mouseCoord, view, rotationAmount); // rotate view clockwise when scrolled upwards
				else if (event.mouseWheelScroll.delta < 0)
					rotateViewAt(mouseCoord, view, -rotationAmount); // rotate view anti-clockwise when scrolled downwards
			}
				break;
			}
		}

		// update the window's view to current state of our custom view
		window.setView(view);

		// render
		window.clear();
		window.draw(rectangle);
		window.display();
	}
}
```
This example displays a white rectangle in the centre of the view, which begins - as usual - in the centre of the window. The view, however, can be rotated using the mouse wheel. The centre of rotation is determined by the current position of the mouse.

**Written by Hapaxia ([Github](http://github.com/hapaxia) | [Twitter](https://twitter.com/Hapaxiation) | [SFML forum](http://en.sfml-dev.org/forums/index.php?action=profile;u=13086))**