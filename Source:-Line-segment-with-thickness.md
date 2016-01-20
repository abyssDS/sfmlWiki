Two different ways to draw a line with thickness between two points.

- Deriving from sf::Drawable
- Deriving from sf::Shape

![Line segment](http://kojirion.github.io/images/segment.png)


**Deriving from sf::Drawable**

A simple class that derives from sf::Drawable only.

```cpp
class sfLine : public sf::Drawable
{
public:
    sfLine(const sf::Vector2f& point1, const sf::Vector2f& point2):
        color(sf::Color::Yellow), thickness(5.f)
    {
        sf::Vector2f direction = point2 - point1;
        sf::Vector2f unitDirection = direction/std::sqrt(direction.x*direction.x+direction.y*direction.y);
        sf::Vector2f unitPerpendicular(-unitDirection.y,unitDirection.x);

        sf::Vector2f offset = (thickness/2.f)*unitPerpendicular;

        vertices[0].position = point1 + offset;
        vertices[1].position = point2 + offset;
        vertices[2].position = point2 - offset;
        vertices[3].position = point1 - offset;

        for (int i=0; i<4; ++i)
            vertices[i].color = color;
    }

    void draw(sf::RenderTarget &target, sf::RenderStates states) const
    {
        target.draw(vertices,4,sf::Quads);
    }


private:
    sf::Vertex vertices[4];
    float thickness;
    sf::Color color;
};
```

**Deriving from sf::Shape**

Header and source for a sf::LineShape, closely modelled after sf::RectangleShape. These can be easily added to one's copy of SFML with minor additions to the relevant CMakeLists.txt and the Graphics master header. Doxygen comments were removed for space.

```cpp
#ifndef SFML_LINESHAPE_HPP
#define SFML_LINESHAPE_HPP

#include <SFML/Graphics/Export.hpp>
#include <SFML/Graphics/Shape.hpp>


namespace sf
{
	class SFML_GRAPHICS_API LineShape : public Shape
	{
		public :

		explicit LineShape(const Vector2f& point1, const Vector2f& point2);

		void setThickness(float thickness);

		float getThickness() const;

		float getLength() const;

		virtual unsigned int getPointCount() const;

		virtual Vector2f getPoint(unsigned int index) const;

		private :

    Vector2f m_direction; ///< Direction of the line
    float m_thickness;    ///< Thickness of the line
};

} // namespace sf


#endif // SFML_LINESHAPE_HPP
```

```cpp
#include <SFML/Graphics/LineShape.hpp>
#include <cmath>


namespace sf
{

LineShape::LineShape(const Vector2f& point1, const Vector2f& point2):
    m_direction(point2 - point1)    
{
    setPosition(point1);
    setThickness(2.f);    
}


void LineShape::setThickness(float thickness)
{
    m_thickness = thickness;
    update();
}


float LineShape::getThickness() const
{
    return m_thickness;
}


float LineShape::getLength() const
{
    return std::sqrt(m_direction.x*m_direction.x+m_direction.y*m_direction.y);
}


unsigned int LineShape::getPointCount() const
{
    return 4;
}


Vector2f LineShape::getPoint(unsigned int index) const
{
    Vector2f unitDirection = m_direction/getLength();
    Vector2f unitPerpendicular(-unitDirection.y,unitDirection.x);

    Vector2f offset = (m_thickness/2.f)*unitPerpendicular;

    switch (index)
    {
        default:
        case 0: return offset;
        case 1: return (m_direction + offset);
        case 2: return (m_direction - offset);
        case 3: return (-offset);
    }
}

} // namespace sf
```