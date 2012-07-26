# A RichText class

[sfe::RichText](https://github.com/Skyrpex/RichText) is an expansion created by Panithadrum (aka Skyrpex) and updated to the latest SFML version by eXpl0it3r. The point is to have something like sf::Text but with the power of having differente styles in it.

Be careful, '\n' characters are not supported. You can use it, but you will see visual errors! (try it)


### Source code

[RichText git repository at Github](https://github.com/Skyrpex/RichText)

### Example

```cpp
#include "RichText.hpp"
#include <SFML/Graphics.hpp>
 
int main()
{
	sf::RichText text;
	text << sf::Text::Bold << sf::Color::Cyan << "This"
		<< sf::Text::Italic << sf::Color::White << " is cool "
		<< sf::Text::Regular << sf::Color::Green << "mate"
		<< sf::Color::White << ". "
		<< sf::Text::Underlined << "I wish I could lick it!";
 
	text.setCharacterSize(25);
	text.setPosition(400, 300);
	text.setOrigin(text.getWidth()/2.f, text.getHeight()/2.f);
 
	sf::RenderWindow window;
	window.create(sf::VideoMode(800, 600), "Rich text");
 
	while(window.isOpen())
	{
		sf::Event event;
		while(window.pollEvent(event))
		{
			if(event.type == sf::Event::Closed)
				window.close();
		}
 
		window.clear();
		window.draw(text);
		window.display();
	}
}
```