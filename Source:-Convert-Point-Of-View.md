# ConvertPointOfView free function

Free function that converts a point from one view's co-ordinate system to another view's co-ordinate system.

Note that this function can also take into account the viewports of the views or ignore them; it's your choice.

Please note that a minimum of C++11 is required for this exact code. If you need non-C++11 and are unable to make the simple minor adaptations, you can [let me know](https://twitter.com/Hapaxiation).

## Function
```c++
sf::Vector2f convertPointOfView(sf::Vector2f point, const sf::View& sourceView, const sf::View& destinationView, bool mapViewports = false)
{
	point = sourceView.getTransform().transformPoint(point);
	if (!mapViewports)
		return destinationView.getInverseTransform().transformPoint(point);
	const sf::FloatRect sourceViewport{ sourceView.getViewport() };
	const sf::FloatRect destinationViewport{ destinationView.getViewport() };
	point.x = ((point.x + 1.f) * sourceViewport.width / 2.f + sourceViewport.left - destinationViewport.left) * 2.f / destinationViewport.width - 1.f;
	point.y = 1.f - (((1.f - point.y) * sourceViewport.height / 2.f + sourceViewport.top - destinationViewport.top) * 2.f / destinationViewport.height);
	return destinationView.getInverseTransform().transformPoint(point);
}
```

## Some Information
First, it's worth noting that you can convert across views by mapping the co-ordinates to a pixel and then mapping that pixel to the second view (using [mapCoordsToPixel and mapPixelToCoords](https://www.sfml-dev.org/tutorials/2.5/graphics-view.php#coordinates-conversions)). However, you lose some accuracy as it quantises to the nearest pixel.

Second, to accurately map across views but not take into account the viewports, it can be done quite simply with one line (this function uses a version of this code if viewports are ignored):  
`sf::Vector2f mappedCoordinate{ mappedView.getInverseTransform().transformPoint(originalView.getTransform().transformPoint(originalCoordinate)) };`  
However, if the viewports of these views are not identical, the co-ordinates will not be in the same position in the window. It maps from "within" the first viewport to within the second viewport.

The function here defaults to ignoring the viewports so effectively just performs the above line of code. You can specify that the viewports are to be considered and it will automatically map across them.

## Usage
To convert the source view's co-ordinate to the destination view's co-ordinate, you specify - as parameters to the function - the co-ordinate for the source view as well as the source view itself and the destination view. The fourth parameter is a boolean that specifies if viewports should be taken into account; this defaults to false (ignores viewports) as the rest of the code is unnecessary unless viewports are used.

`sf::Vector2f destinationCoord{ convertPointOfView(sourceCoord, sourceView, desinationView) }; // ignores viewports`  
`sf::Vector2f destinationCoord{ convertPointOfView(sourceCoord, sourceView, desinationView, true) }; // uses viewports`  

## Example
Here's a full example, which also includes the function:

```c++
#include <SFML/Graphics.hpp>
#include <iostream>

sf::Vector2f convertPointOfView(sf::Vector2f point, const sf::View& sourceView, const sf::View& destinationView, bool mapViewports = false)
{
	point = sourceView.getTransform().transformPoint(point);
	if (!mapViewports)
		return destinationView.getInverseTransform().transformPoint(point);
	const sf::FloatRect sourceViewport{ sourceView.getViewport() };
	const sf::FloatRect destinationViewport{ destinationView.getViewport() };
	point.x = ((point.x + 1.f) * sourceViewport.width / 2.f + sourceViewport.left - destinationViewport.left) * 2.f / destinationViewport.width - 1.f;
	point.y = 1.f - (((1.f - point.y) * sourceViewport.height / 2.f + sourceViewport.top - destinationViewport.top) * 2.f / destinationViewport.height);
	return destinationView.getInverseTransform().transformPoint(point);
}

int main()
{
	const sf::Vector2u windowSize{ 960u, 540u };
	const std::string windowTitle{ "Convert Point of View" };

	// VIEWS
	sf::View view1;
	view1.setSize(sf::Vector2f(windowSize)); // (0, 0) - (960, 540)
	view1.setCenter(sf::Vector2f(windowSize / 2u)); // (480, 270)
	sf::View view2(view1);
	view2.setSize(sf::Vector2f(windowSize / 2u)); // (0, 0) - (480, 270)
	view2.setCenter(sf::Vector2f(windowSize / 4u)); // (240, 135)
	view1.setViewport({ { 0.f, 0.f },{ 0.75f, 1.f } });
	view2.setViewport({ { 0.1f, 0.25f },{ 0.9f, 0.5f } });

	// VIEWPORT RECTANGLES (to make their region visible)
	sf::RectangleShape viewportRectangle1;
	sf::RectangleShape viewportRectangle2;
	viewportRectangle1.setFillColor(sf::Color(0, 64, 0));
	viewportRectangle2.setFillColor(sf::Color(64, 0, 0));
	viewportRectangle1.setSize(view1.getSize());
	viewportRectangle1.setOrigin(view1.getSize() / 2.f);
	viewportRectangle1.setPosition(view1.getCenter());
	viewportRectangle2.setSize(view2.getSize());
	viewportRectangle2.setOrigin(view2.getSize() / 2.f);
	viewportRectangle2.setPosition(view2.getCenter());

	// CIRCLES (to show the point in each view)
	sf::CircleShape circle1(5.f);
	circle1.setOrigin({ circle1.getRadius(), circle1.getRadius() });
	circle1.setPosition({ 480.f, 270.f });
	sf::CircleShape circle2(circle1);
	circle1.setFillColor(sf::Color::Green);
	circle2.setFillColor(sf::Color::Red);

	bool mapViewports{ false };
	sf::RenderWindow window(sf::VideoMode(windowSize.x, windowSize.y), windowTitle);
	while (window.isOpen())
	{
		sf::Event event;
		while (window.pollEvent(event))
		{
			if (event.type == sf::Event::Closed)
				window.close();
			else if ((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::M))
			{
				mapViewports = !mapViewports;
				window.setTitle(windowTitle + (mapViewports ? + " (mapping across viewports)" : ""));
			}
		}
		sf::Vector2f position{ window.mapPixelToCoords(sf::Mouse::getPosition(window), view1) };
		circle1.setPosition(position);

		circle2.setPosition(convertPointOfView(circle1.getPosition(), view1, view2, mapViewports)); // this is the important line!

		window.clear();
		window.setView(view1);
		window.draw(viewportRectangle1);
		window.draw(circle1);
		window.setView(view2);
		window.draw(viewportRectangle2, sf::BlendAdd);
		window.draw(circle2, sf::BlendAdd);
		window.display();
	}
}
```
This example creates two views, each with its own viewport. A rectangle is shown in each viewport than fills the entire viewport. View/viewport 1 is green and view/viewport 2 is red; they overlap. The green view is considered the source view and the red view is considered the destination view. It also shows two "circles" (actually elliptical due to the views) that represent the source co-ordinate (green) and the destination co-ordinate (red). The source co-ordinate in this example follows the mouse cursor (even, actually, when outside of the source viewport!)

When the program starts, the viewports are ignored during co-ordinate mapping. This shows the green circle following the mouse but the red circle is in a position within the red rectangle as the green circle is within the green rectangle; the co-ordinate is converted from one view to another.
To map across viewports, you can press M on the keyboard (this toggles mapping across viewports). When this is active, you will notice that the red circle is now in an identical place in the window to the green circle (under the mouse cursor) although the green circle cannot be displayed outside of the green rectangle (viewport 1) and the red circle cannot be displayed outside of the red rectangle (viewport 2).

**Written by Hapaxia ([Github](http://github.com/hapaxia) | [Twitter](https://twitter.com/Hapaxiation) | [SFML forum](http://en.sfml-dev.org/forums/index.php?action=profile;u=13086))**