# Using some sort of RichText

sf::RichText is an expansion created by Panithadrum and updated to the lasted SFML version by eXpl0it3r. The point is to have something like sf::Text but with the power of having differente styles in it.

Be careful, '\n' characters are not supported. You can use it, but you will see visual errors! (try it)

### Example.cpp

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


### RichText.hpp

```cpp
#pragma once
 
//////////////////////////////////////////////////////////////////////////
// Headers
//////////////////////////////////////////////////////////////////////////
#include <SFML/Graphics/Drawable.hpp>
#include <SFML/Graphics/Color.hpp>
#include <SFML/Graphics/Text.hpp>
#include <SFML/System/String.hpp>
#include <list>
 
namespace sf
{
	class RichText : public sf::Drawable, public sf::Transformable
	{
	public:
		//////////////////////////////////////////////////////////////////////////
		// Constructor
		//////////////////////////////////////////////////////////////////////////
		RichText();
 
		//////////////////////////////////////////////////////////////////////////
		// Operators
		//////////////////////////////////////////////////////////////////////////
		RichText & operator << (const sf::Color &color);
		RichText & operator << (sf::Text::Style style);
		RichText & operator << (const sf::String &string);
 
		//////////////////////////////////////////////////////////////////////////
		// Set character size
		//////////////////////////////////////////////////////////////////////////
		void setCharacterSize(unsigned int size);
 
		//////////////////////////////////////////////////////////////////////////
		// Set font
		//////////////////////////////////////////////////////////////////////////
		void setFont(const sf::Font &font);
 
		//////////////////////////////////////////////////////////////////////////
		// Clear
		//////////////////////////////////////////////////////////////////////////
		void clear();
 
		//////////////////////////////////////////////////////////////////////////
		// Get text list
		//////////////////////////////////////////////////////////////////////////
		const std::list<sf::Text> &getTextList() const;
 
		//////////////////////////////////////////////////////////////////////////
		// Get character size
		//////////////////////////////////////////////////////////////////////////
		unsigned int getCharacterSize() const;
 
		//////////////////////////////////////////////////////////////////////////
		// Get font
		//////////////////////////////////////////////////////////////////////////
		const sf::Font & getFont() const;
 
		//////////////////////////////////////////////////////////////////////////
		// Get width
		//////////////////////////////////////////////////////////////////////////
		float getWidth() const;
 
		//////////////////////////////////////////////////////////////////////////
		// Get height
		//////////////////////////////////////////////////////////////////////////
		float getHeight() const;
 
	private:
		//////////////////////////////////////////////////////////////////////////
		// Update size
		//////////////////////////////////////////////////////////////////////////
		void updateSize() const;
 
		//////////////////////////////////////////////////////////////////////////
		// Update position
		//////////////////////////////////////////////////////////////////////////
		void updatePosition() const;
 
		//////////////////////////////////////////////////////////////////////////
		// Render
		//////////////////////////////////////////////////////////////////////////
		void draw(RenderTarget& target, RenderStates states) const;
 
		//////////////////////////////////////////////////////////////////////////
		// Member data
		//////////////////////////////////////////////////////////////////////////
		mutable std::list<sf::Text> myTexts;	///< List of texts
		sf::Color myCurrentColor;				///< Last used color
		sf::Text::Style myCurrentStyle;			///< Last style used
		mutable sf::Vector2f mySize;			///< Size of the text
		mutable bool mySizeUpdated;				///< Do we need to recompute the size ?
		mutable bool myPositionUpdated;			///< Do we need to recompute the position ?
	};
}
```


### RichText.cpp

```cpp
//////////////////////////////////////////////////////////////////////////
// Headers
//////////////////////////////////////////////////////////////////////////
#include "RichText.hpp"
#include <Sfml/Graphics/RenderTarget.hpp>
#include <iostream>
 
namespace sf
{
	RichText::RichText()
		: myCurrentColor(sf::Color::White), myCurrentStyle(sf::Text::Regular), mySizeUpdated(false), myPositionUpdated(false)
	{
 
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Operator << sf::Color
	//////////////////////////////////////////////////////////////////////////
	RichText & RichText::operator << (const sf::Color &color)
	{
		myCurrentColor = color;
		return *this;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Operator << sf::Text::Style
	//////////////////////////////////////////////////////////////////////////
	RichText & RichText::operator << (sf::Text::Style style)
	{
		myCurrentStyle = style;
		return *this;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Operator << sf::String
	//////////////////////////////////////////////////////////////////////////
	/*
	**	Must parse the strings to look for '\n' characters. If found, we break
	**	the string into two pieces.
	*/
	RichText & RichText::operator << (const sf::String &string)
	{
		// It is not updated
		mySizeUpdated = false;
		myPositionUpdated = false;
 
		// Find \n characters (assert)
		//assert(string.Find('\n') == std::string::npos);
		if(string.find('\n') != std::string::npos) std::cerr << "sf::RichtText: Oops, character \\n found. You will get visual errors." << std::endl;
 
		// Add string
		myTexts.resize(myTexts.size()+1);
 
		// Setup string
		sf::Text &text = *(--myTexts.end());
		text.setColor(myCurrentColor);
		text.setStyle(myCurrentStyle);
		text.setString(string);
 
		// Return
		return *this;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Set size
	//////////////////////////////////////////////////////////////////////////
	void RichText::setCharacterSize(unsigned int size)
	{
		// Set character size
		for(std::list<sf::Text>::iterator it = myTexts.begin(); it != myTexts.end(); ++it)
		{
			it->setCharacterSize(size);
		}
 
		// It is not updated
		mySizeUpdated = false;
		myPositionUpdated = false;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Set font
	//////////////////////////////////////////////////////////////////////////
	void RichText::setFont(const sf::Font &font)
	{
		// Set character size
		for(std::list<sf::Text>::iterator it = myTexts.begin(); it != myTexts.end(); ++it)
		{
			it->setFont(font);
		}
 
		// It is not updated
		mySizeUpdated = false;
		myPositionUpdated = false;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Clear
	//////////////////////////////////////////////////////////////////////////
	void RichText::clear()
	{
		// Clear text list
		myTexts.clear();
 
		// Reset size
		mySize = sf::Vector2f(0.f, 0.f);
 
		// It is updated
		mySizeUpdated = true;
		myPositionUpdated = true;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Get text list
	//////////////////////////////////////////////////////////////////////////
	const std::list<sf::Text> & RichText::getTextList() const
	{
		return myTexts;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Get character size
	//////////////////////////////////////////////////////////////////////////
	unsigned int RichText::getCharacterSize() const
	{
		if(myTexts.size()) return myTexts.begin()->getCharacterSize();
		return 0;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Get font
	//////////////////////////////////////////////////////////////////////////
	const sf::Font & RichText::getFont() const
	{
		if(myTexts.size()) return myTexts.begin()->getFont();
		return sf::Font::getDefaultFont();
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Get width
	//////////////////////////////////////////////////////////////////////////
	float RichText::getWidth() const
	{
		updateSize();
		return mySize.x;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Get height
	//////////////////////////////////////////////////////////////////////////
	float RichText::getHeight() const
	{
		updateSize();
		return mySize.y;
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Render
	//////////////////////////////////////////////////////////////////////////
	void RichText::draw(RenderTarget& target, RenderStates states) const
	{
		// Update position
		updatePosition();

		states.transform *= getTransform();
 
		// Draw
		for(std::list<sf::Text>::const_iterator it = myTexts.begin(); it != myTexts.end(); ++it)
		{
			// Add transformation
			//it->setT

			// Draw text
			target.draw(*it, states);
		}
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Update size
	//////////////////////////////////////////////////////////////////////////
	void RichText::updateSize() const
	{
		// Return if updated
		if(mySizeUpdated) return;
 
		// Return if empty
		if(myTexts.begin() == myTexts.end()) return;
 
		// It is updated
		mySizeUpdated = true;
 
		// Sum all sizes (height not implemented)
		mySize.x = 0.f;
		mySize.y = myTexts.begin()->getGlobalBounds().height;
		for(std::list<sf::Text>::const_iterator it = myTexts.begin(); it != myTexts.end(); ++it)
		{
			// Update width
			mySize.x += it->getGlobalBounds().width;
		}
	}
 
	//////////////////////////////////////////////////////////////////////////
	// Update position
	//////////////////////////////////////////////////////////////////////////
	void RichText::updatePosition() const
	{
		// Return if updated
		if(myPositionUpdated) return;
 
		// Return if empty
		if(myTexts.begin() == myTexts.end()) return;
 
		// It is updated
		myPositionUpdated = true;
 
		// Get starting position
		sf::Vector2f offset;
 
		// Draw
		for(std::list<sf::Text>::iterator it = myTexts.begin(); it != myTexts.end(); ++it)
		{
			// Set all the origins to the first one
			it->setOrigin(it->getPosition() - myTexts.begin()->getPosition() - offset);
 
			// Set offset
			const sf::FloatRect rect = it->getGlobalBounds();
			offset.x += rect.width;
		}
	}
}
```