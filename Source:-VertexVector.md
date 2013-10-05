**Extremely easy to use, straightforward alternative to VertexArray**

Supports emplacement (performance win!) and foreach loops

```cpp
#ifndef VERTEXVECTOR
#define VERTEXVECTOR

#include <vector>
#include <SFML/Graphics.hpp>

template<sf::PrimitiveType TPrimitive> struct VertexVector : public std::vector<sf::Vertex>, public sf::Drawable
{
    using std::vector<sf::Vertex>::vector;
    inline void draw(sf::RenderTarget& mRenderTarget, sf::RenderStates mRenderStates) const
    {
        mRenderTarget.draw(&this->operator [](0), this->size(), TPrimitive, mRenderStates);
    }
};

#endif
```

Usage example:

```cpp
sf::RenderTarget renderTarget; // Example rendertarget
VertexVector<sf::PrimitiveType::Quads> vertices;

vertices.emplace_back(sf::Vector2f(1.f, 1.f), sf::Color::Red);
vertices.emplace_back(sf::Vector2f(10.f, 1.f), sf::Color::Red);
vertices.emplace_back(sf::Vector2f(10.f, 10.f), sf::Color::Red);
vertices.emplace_back(sf::Vector2f(1.f, 10.f), sf::Color::Red);

for(auto& v : vertices) v.color = sf::Color::Green;
renderTarget.draw(vertices);
```