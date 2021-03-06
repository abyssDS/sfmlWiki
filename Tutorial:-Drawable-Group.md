# Grouping your Drawable in a Group class

_This page lacks examples and images._

This tutorial has been written because of [this thread](http://fr.sfml-dev.org/forums/index.php?topic=7719.new#new).

## Introduction

Drawable's children cover a wide range of uses: Shapes, Sprites, Texts, etc. but the "family" lacks one member: Group, so we'll create it there. Indeed, the class exists in well-known libraries such as [Pygame](http://www.pygame.org/news.html).

## Utility

### Use of std::vector

Why are Groups useful? Basically, you can store your game objects in them so that they don't "swim" in a big "pool", and apply precise calculations on a specified part of them easily.
* You can make a "backgrounds" Group, followed by a "Level" one, and then a "Foreground" One, for example. This, way, backgrounds will always be drawn before the level tiles which will also be drawn before the foreground elements, even if you had backgrounds, tiles, or foregrounds, which would have otherwise been on the top. Imagine you add a background without using Groups: it covers everything and you can't see behind it!
* You can make a traditionnal 2D game with depth, where there are two parallels level. The character must only be on one of the layers, and change by jumping. Each "layer" can be implemented as a Group, and the one where the player stands is used for collisions, while the other can be tinted in grey (`Group.SetColor(sf::Color(100, 100, 100))`) to show that the player is standing on the other side.
* You'll find many others.

### Inheritance from Drawable

Group would behave like a simple [std::vector](http://en.cppreference.com/w/cpp/container/vector), but would also be drawable itself. There are two main reasons for this:
* simplicity: it's easier to type `Group.Draw()` than `for(int i = 0; ...) { Group[i].Draw() }`. It also makes Groups storable in other Groups, like other Drawables, which ables to process the whole very easily ;
* sharing of properties for the membres of a Group. For example, if you want all the Sprites stored in a Group to move at the same time, you can just update the coordinates of the Group. This is very useful if you make an articulated body made of sprites. You can also separate objects around a single point: the origin of the Group, just by using `Group.SetCenter(..., ...)`!

## Suggested implementation (using SFML 2.x & C++11)

Here's what I use. The SFML doesn't implement a Group class for ownership reasons: should the Group destroy its elements when it is destroyed? etc.

### Group.hpp
```cpp
#ifndef GROUP_INCLUDED_HPP
#define GROUP_INCLUDED_HPP

#include <SFML/Graphics.hpp>
#include <vector>
#include <functional>

class Group : public sf::Drawable {
public:
    Group();
    virtual ~Group() = default;

    virtual void draw(sf::RenderTarget& target, sf::RenderStates states) const;

    const sf::Drawable& operator[](std::size_t index);
    std::size_t push_back(const sf::Drawable& drawable);
    const sf::Drawable& pop_back();

private:
    std::vector<std::reference_wrapper<const sf::Drawable>> m_drawables;
};

#endif
```

### Group.cpp
```cpp
#include "Group.hpp"

Group::Group()
: m_drawables{} {

}

void Group::draw(sf::RenderTarget& target, sf::RenderStates states) const {
    for(const auto& drawable : m_drawables) {
        target.draw(drawable, states);
    }
}

const sf::Drawable& Group::operator[](std::size_t index) {
    return m_drawables[index];
}

std::size_t Group::push_back(const sf::Drawable& drawable) {
    m_drawables.push_back(drawable);
    return m_drawables.size() - 1;
}

const sf::Drawable& Group::pop_back() {
    const auto& drawable = m_drawables.back();
    m_drawables.pop_back();
    return drawable;
}
```

## Examples

_It would be a good idea to use the "reasons" said above and make code snippets out of them_