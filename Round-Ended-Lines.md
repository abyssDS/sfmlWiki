## Summary
This is a class for SFML 2 that provides an easy interface to lines with rounded ends.

Class written by Foaly. You can use it in any way you want. Just a little note that the class was written by me would be nice :)


## Source
### Header File
```cpp
#ifndef ROUNDENDEDLINE_H
#define ROUNDENDEDLINE_H

// Class written by Foaly

#include <SFML/Graphics/Shape.hpp>
#include <cmath>


class CRoundendedLine : public sf::Shape
{
public:

    CRoundendedLine(const sf::Vector2f& endPoint = sf::Vector2f(0, 0), const float width = 1.0);

    void setEndPoint(const sf::Vector2f& endPoint);

    void setWidth(const float width);

    virtual unsigned int getPointCount() const;

    virtual sf::Vector2f getPoint(unsigned int index) const;

private :

    sf::Vector2f m_endPoint;
    float m_Width;
};

#endif //ROUNDENDEDLINE_H

```

### Implementation
```
#include "RoundendedLine.h"

CRoundendedLine::CRoundendedLine(const sf::Vector2f& endPoint, const float width) : m_endPoint (endPoint), m_Width (width)
{
    update();
}

void CRoundendedLine::setEndPoint(const sf::Vector2f& endPoint)
{
    m_endPoint = endPoint;
    update();
}

void CRoundendedLine::setWidth(const float width)
{
    m_Width = width;
    update();
}

unsigned int CRoundendedLine::getPointCount() const
{
    return 30;
}

sf::Vector2f CRoundendedLine::getPoint(unsigned int index) const
{
    sf::Vector2f P1(1.0, 0.0);
    sf::Vector2f P2(m_endPoint + sf::Vector2f(1.0, 0.0) - getPosition());

    sf::Vector2f offset;
    int iFlipDirection;

    if(index < 15)
    {
        offset = P2;
        iFlipDirection = 1;
    }
    else
    {
        offset = P1;
        iFlipDirection = -1;
        index -= 15;
    }

    float start = -atan2(P1.y - P2.y, P2.x - P1.x);

    float angle = index * M_PI / 14 - M_PI / 2 + start;
    float x = std::cos(angle) * m_Width / 2;
    float y = std::sin(angle) * m_Width / 2;

    return sf::Vector2f(offset.x + x * iFlipDirection, offset.y + y * iFlipDirection);
}
```
## Example Usage
This is a small example of how the class works.
```cpp
#include <SFML/Graphics.hpp>
#include "RoundendedLine.h"


int main()
{
    sf::RenderWindow window(sf::VideoMode(500, 500), "Round Ended Line Example");

    CRoundendedLine RoundendedLine;
    RoundendedLine.setPosition(200, 200);
    RoundendedLine.setEndPoint(sf::Vector2f(300, 300));
    RoundendedLine.setWidth(10);
    RoundendedLine.setFillColor(sf::Color(sf::Color::Red));
    RoundendedLine.setOutlineThickness(2);
    RoundendedLine.setOutlineColor(sf::Color(sf::Color::Yellow));


    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            // Close Event
            if (event.type == sf::Event::Closed)
                window.close();

            // Escape key pressed
            if ((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Escape))
                window.close();
        }


        window.clear();
        window.draw(RoundendedLine);
        window.display();
    }

    return 0;
}
```
Here is an image of what the sample code looks like in action:

![Eample](http://img6.imagebanana.com/img/jie05egl/Unbenannt.png)