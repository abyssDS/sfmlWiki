# RadialGradient Shader
## Introduction
This is a simple and basic shader that allows you to add a radial gradient to any untextured SFML graphical object.
It is a fragment shader and can be used with the default vertex shader (by not loading one) or using the one provided here.

## Shader
The fragment shader:
```c++
const char RadialGradient[] =
"uniform vec4 color;"
"uniform float expand;"
"uniform vec2 center;"
"uniform float radius;"
"uniform float windowHeight;"
"void main(void)"
"{"
"vec2 centerFromSfml = vec2(center.x, windowHeight - center.y);"
"vec2 p = (gl_FragCoord.xy - centerFromSfml) / radius;"
	"float r = sqrt(dot(p, p));"
	"if (r < 1.0)"
	"{"
		"gl_FragColor = mix(color, gl_Color, (r - expand) / (1 - expand));"
	"}"
	"else"
	"{"
		"gl_FragColor = gl_Color;"
	"}"
"}";
```

A default vertex shader if you require one:
```c++
const char VertexShader[] =
"void main()"
"{"
	"gl_Position = gl_ModelViewProjectionMatrix * gl_Vertex;"
	"gl_TexCoord[0] = gl_TextureMatrix[0] * gl_MultiTexCoord0;"
	"gl_FrontColor = gl_Color;"
"}";
```
Be aware that if you use a custom vertex shader, gl_Color will need to be passed on as above.

## Parameters
The shader takes five parameters:
* **color** (sf::Color)  
the inner colour of the gradient
* **center** (sf::Vector2f)  
the position of the centre of the gradient
* **radius** (float)  
the radius of the gradient (length from centre of gradient to outer colour)
* **expand** (float)  
an alpha value (0.f - 1.f) that specifies how much of that radius in only inner colour (the gradient starts outside of that amount)
* **windowHeight** (float)  
the height of the window being drawn to. This is required.

**Important: The position and radius given _must_ be in window pixel co-ordinates.**

## Usage
Assuming that we have a sf::CircleShape set up like this:
```c++
sf::CircleShape circle;
circle.setRadius(100.f);
circle.setOrigin(circle.getRadius(), circle.getRadius());
circle.setPosition(sf::Vector2f(window.getSize()) / 2.f);
circle.setFillColor(sf::Color::White);
```
which will display a (white) circle in the centre of the window with a diameter of 200.

We can use:
```c++
shader.loadFromMemory(VertexShader, RadialGradient);
shader.setParameter("color", sf::Color::Blue);
shader.setParameter("center", circle.getPosition());
shader.setParameter("radius", circle.getRadius());
shader.setParameter("expand", 0.f);
shader.setParameter("windowHeight", static_cast<float>(window.getSize().y)); // this must be set, but only needs to be set once (or whenever the size of the window changes)
```
which loads the shader and tells it where the gradient should be.
In this case, the radial gradient is the size of the circle and starts from the centre of it.

Here are screenshots showing the effect of the **expand** parameter, which expands the inner colour:
![expand = 0.f](http://i.imgur.com/EiBTeoJ.png) expand = 0.f  
![expand = 0.25f](http://i.imgur.com/8IG5FFE.png) expand = 0.25f  
![expand = 0.5f](http://i.imgur.com/VpFdCzL.png) expand = 0.5f  
![expand = 0.75f](http://i.imgur.com/jFw76Dk.png) expand = 0.75f  
![expand = 1.f](http://i.imgur.com/jURnd4P.png) expand = 1.f

The fill colour of the circle become the outer colour of the radial gradient and the inner colour of the gradient is passed as a parameter to the shader. This way, if you radial gradient is in a position outside of the object, the object is displayed as normal (using the fill colour).

Note: the position given for the centre of the gradient is a global position, not an object's local position. If you move an object, and you want the gradient to follow, you need to update this parameter.

## Example
Here is a complete example, which also includes the shader code:
```c++
#include <SFML/Graphics.hpp>
#include <iostream>

const char VertexShader[] =
"void main()"
"{"
	"gl_Position = gl_ModelViewProjectionMatrix * gl_Vertex;"
	"gl_TexCoord[0] = gl_TextureMatrix[0] * gl_MultiTexCoord0;"
	"gl_FrontColor = gl_Color;"
"}";

const char RadialGradient[] =
"uniform vec4 color;"
"uniform vec2 center;"
"uniform float radius;"
"uniform float expand;"
"uniform float windowHeight;"
"void main(void)"
"{"
"vec2 centerFromSfml = vec2(center.x, windowHeight - center.y);"
"vec2 p = (gl_FragCoord.xy - centerFromSfml) / radius;"
	"float r = sqrt(dot(p, p));"
	"if (r < 1.0)"
	"{"
		"gl_FragColor = mix(color, gl_Color, (r - expand) / (1 - expand));"
	"}"
	"else"
	"{"
		"gl_FragColor = gl_Color;"
	"}"
"}";

int main()
{
	if (!sf::Shader::isAvailable())
	{
		std::cerr << "Shaders are not available." << std::endl;
		return EXIT_FAILURE;
	}

	sf::RenderWindow window(sf::VideoMode(800, 600), "Radial Gradient", sf::Style::Default);
	window.setFramerateLimit(30);

	sf::CircleShape circle;
	sf::RectangleShape rectangle;
	sf::Shader shader;

	circle.setRadius(100.f);
	circle.setOrigin(circle.getRadius(), circle.getRadius());
	circle.setPosition(sf::Vector2f(window.getSize()) / 2.f);
	circle.setFillColor(sf::Color::Transparent);

	rectangle.setSize({ 300.f, 200.f });
	rectangle.setFillColor(sf::Color::Yellow);
	rectangle.setOrigin(rectangle.getLocalBounds().width / 2.f, rectangle.getLocalBounds().height / 2.f);
	rectangle.setPosition(sf::Vector2f(window.getSize()) / 4.f);

	shader.loadFromMemory(VertexShader, RadialGradient);
	shader.setParameter("windowHeight", static_cast<float>(window.getSize().y)); // this must be set, but only needs to be set once (or whenever the size of the window changes)

	while (window.isOpen())
	{
		sf::Event event;
		while (window.pollEvent(event))
		{
			if (event.type == sf::Event::Closed || event.type == sf::Event::KeyPressed && event.key.code == sf::Keyboard::Escape)
				window.close();
		}

		window.clear();
		shader.setParameter("color", sf::Color::Red);
		shader.setParameter("center", rectangle.getPosition() + sf::Vector2f(100.f, 100.f));
		shader.setParameter("radius", 200.f);
		shader.setParameter("expand", 0.25f);
		window.draw(rectangle, &shader);
		shader.setParameter("color", sf::Color::Blue);
		shader.setParameter("center", circle.getPosition());
		shader.setParameter("radius", circle.getRadius());
		shader.setParameter("expand", 0.f);
		window.draw(circle, &shader);
		window.display();
	}

	return EXIT_SUCCESS;
}
```
This example displays:
![example](http://i.imgur.com/13Q4M1Z.png)

**Written by Hapax ([Github](http://github.com/hapaxia) | [SFML forum](http://en.sfml-dev.org/forums/index.php?action=profile;u=13086))**