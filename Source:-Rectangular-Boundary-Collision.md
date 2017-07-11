# Introduction
SFML's sprites and shapes can provide you with their axis-aligned boundary boxes (getGlobalBounds) and you can easily use these rectangles to test if they intersect. Unfortunately, this _only_ allows axis-aligned collision detection.

Rectangular Boundary Collision allows testing collision based on their original rectangular boundary box (getLocalBounds).
It tests to see if and object's rectangular boundary collides with another object's rectangular boundary. This method, however, allows transformations to be taken into account - most notably, rotation.

Rectangular Boundary Collision is a single, templated free-function contained within the "collision" namespace:  
```cpp
const bool spritesAreColliding = collision::areColliding(sprite1, sprite2);
```

# Code
```cpp
#ifndef RECTANGULAR_BOUNDARY_COLLISION_HPP
#define RECTANGULAR_BOUNDARY_COLLISION_HPP

#include <SFML/Graphics.hpp>
#include <array>

namespace collision
{
    namespace impl
    {

bool satRectangleAndPoints(const sf::Vector2f rectangleSize, const std::array<sf::Vector2f, 4>& points);

    } // namespace impl

// returns a boolean representing if the two objects' rectangular boundaries are colliding
// allows the collision level to be specified:
//     0: AABB (axis-aligned boundaries boxes)
//     1: tests if any corners are inside opposite rectangle (performs level 0 first)
//     2: SAT (separating axis theorem) (performs levels 0 and 1 first)
//    -1: maximum level (default) (performs levels 0, 1 and 2)
// "short-circuits" higher level tests if a lower level is certain of a result
// allows a negative value for collision level to represent the "maximum" collision level (default value)
// can test any two (different) objects as long as they have these functions:
//     getTransform()        (must return sf::Transform)
//     getInverseTransform() (must return sf::Transform)
//     getLocalBounds()      (must return sf::FloatRect)
template <class T1, class T2>
bool areColliding(const T1& object1, const T2& object2, const int collisionLevel = -1)
{
    // LEVEL 0 (axis-aligned bounding box)
    const bool level0{ object1.getGlobalBounds().intersects(object2.getGlobalBounds()) };
    if (!level0 || collisionLevel == 0)
        return level0;

    // LEVEL 1 (any corners inside opposite rectangle)
    const sf::Transform transform1{ object1.getTransform() };
    const sf::Transform transform2{ object2.getTransform() };
    const sf::Transform inverseTransform1{ object1.getInverseTransform() };
    const sf::Transform inverseTransform2{ object2.getInverseTransform() };
    const sf::FloatRect rect1Bounds{ object1.getLocalBounds() };
    const sf::FloatRect rect2Bounds{ object2.getLocalBounds() };
    const sf::Vector2f rect1Size{ rect1Bounds.width, rect1Bounds.height };
    const sf::Vector2f rect2Size{ rect2Bounds.width, rect2Bounds.height };
    const sf::Vector2f rect1TopLeft{ inverseTransform2.transformPoint(transform1.transformPoint({ 0.f, 0.f })) };
    const sf::Vector2f rect1TopRight{ inverseTransform2.transformPoint(transform1.transformPoint({ rect1Size.x, 0.f })) };
    const sf::Vector2f rect1BottomRight{ inverseTransform2.transformPoint(transform1.transformPoint(rect1Size)) };
    const sf::Vector2f rect1BottomLeft{ inverseTransform2.transformPoint(transform1.transformPoint({ 0.f, rect1Size.y })) };
    const sf::Vector2f rect2TopLeft{ inverseTransform1.transformPoint(transform2.transformPoint({ 0.f, 0.f })) };
    const sf::Vector2f rect2TopRight{ inverseTransform1.transformPoint(transform2.transformPoint({ rect2Size.x, 0.f })) };
    const sf::Vector2f rect2BottomRight{ inverseTransform1.transformPoint(transform2.transformPoint(rect2Size)) };
    const sf::Vector2f rect2BottomLeft{ inverseTransform1.transformPoint(transform2.transformPoint({ 0.f, rect2Size.y })) };
    const bool level1{ (
        (rect1Bounds.contains(rect2TopLeft)) ||
        (rect1Bounds.contains(rect2TopRight)) ||
        (rect1Bounds.contains(rect2BottomLeft)) ||
        (rect1Bounds.contains(rect2BottomRight)) ||
        (rect2Bounds.contains(rect1TopLeft)) ||
        (rect2Bounds.contains(rect1TopRight)) ||
        (rect2Bounds.contains(rect1BottomLeft)) ||
        (rect2Bounds.contains(rect1BottomRight))) };
    if (level1 || collisionLevel == 1)
        return level1;

    // LEVEL 2 (SAT)
    std::array<sf::Vector2f, 4> rect1Points
    {
        rect1BottomLeft,
        rect1BottomRight,
        rect1TopRight,
        rect1TopLeft,
    };
    if (!impl::satRectangleAndPoints(rect2Size, rect1Points))
        return false;
    std::array<sf::Vector2f, 4> rect2Points
    {
        rect2BottomLeft,
        rect2BottomRight,
        rect2TopRight,
        rect2TopLeft,
    };
    return impl::satRectangleAndPoints(rect1Size, rect2Points);
}



    namespace impl
    {

bool satRectangleAndPoints(const sf::Vector2f rectangleSize, const std::array<sf::Vector2f, 4>& points)
{
    bool allPointsLeftOfRectangle{ true };
    bool allPointsRightOfRectangle{ true };
    bool allPointsAboveRectangle{ true };
    bool allPointsBelowRectangle{ true };
    for (auto& point : points)
    {
        if (point.x >= 0.f)
            allPointsLeftOfRectangle = false;
        if (point.x <= rectangleSize.x)
            allPointsRightOfRectangle = false;
        if (point.y >= 0.f)
            allPointsAboveRectangle = false;
        if (point.y <= rectangleSize.y)
            allPointsBelowRectangle = false;
    }
    return !(allPointsLeftOfRectangle || allPointsRightOfRectangle || allPointsAboveRectangle || allPointsBelowRectangle);
}

    } // namespace impl
} // namespace collision
#endif // RECTANGULAR_BOUNDARY_COLLISION_HPP
```

# Usage
## Header
To use the code above, you can simply save it as a header file and include it as necessary; it has include guards.

If you would prefer to have the code directly included in another file (header or source), you can remove the include guards (lines starting with `#ifdef`, `#define` and `#endif`)

## Function
The function to use is "areColliding" and it returns a boolean that represents if the two objects are colliding.

```cpp
template <class T1, class T2>
bool areColliding(const T1& object1, const T2& object2, const int collisionLevel = -1);
```

The two objects should be provided as (the first two) parameters to the function; the two object types can be different.
> Note that any objects can be used as long as they have these four functions: getLocalBounds (must return sf::FloatRect), getTransform and getInverseTransform (both must return sf::Transform).  
> Also note that it uses the transformed local boundary box. This means that it also tests the rectangular boundary of a circle, not the circular region.

The third parameter is optional. It specifies the required "collision level".

## Collision Level
The collision is performed in three stages:

- Level 0 - AABB (axis-aligned boundaries boxes):  
tests the axis-aligned boundaries boxes (equivalent to getGlobalBounds) of the objects to see if they intersect.

- Level 1 - Corner inside opposite object:  
tests to see if any of either object's corner is inside the other object.

- Level 2 - SAT (Separating Axis Theorem):  
tests using a simplified version (for rectangles) of SAT to catch other cases.

If, at any time, the result is certain, the following stages are not calculated. This allows easier detections to be done without resorting to the more involved calculations.

The collision level represents after which stage to leave the function, regardless of if a result is certain.

If a negative value is provided for Collision Level (its default value), the maximum level (2) is used.

![Collision Types](http://i.imgur.com/llRqULb.png)  
The screenshots' results are using maximum level (collision level 2).

Therefore, collision results (in order) for level 2:  
false, true, true, true, true
> accurate results

Collision results for level 1 would be:  
false, true, false, true, true
> the third one fails because none of the corners are inside the object - SAT (level 2) is required to clarify this result

Collision results for level 0 would be:  
true, true, true, true, true
> the axis-aligned boundary boxes intersect in all five images resulting in the first one being considered a collision

# License
This code is from https://github.com/Hapaxia/SfmlSnippets/tree/master/RectangularBoundaryCollision and [provided under the zlib license](https://github.com/Hapaxia/SfmlSnippets/blob/master/RectangularBoundaryCollision/LICENSE.txt).

It is from the GitHub repository [SfmlSnippets](https://github.com/Hapaxia/SfmlSnippets) by [Hapaxia](https://github.com/Hapaxia).

# Example
Here is a full example, showing two sprites/rectangles - one of which can be moved/rotated/scaled. The object is coloured red when they collide and coloured green when they do not. This example was used to create the screenshots above.
```cpp
////////////////////////////////////////////////////////////////
//
// The MIT License (MIT)
//
// Copyright (c) 2017 M.J.Silk
//
// Permission is hereby granted, free of charge, to any person obtaining a copy
// of this software and associated documentation files (the "Software"), to deal
// in the Software without restriction, including without limitation the rights
// to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
// copies of the Software, and to permit persons to whom the Software is
// furnished to do so, subject to the following conditions:
//
// The above copyright notice and this permission notice shall be included in all
// copies or substantial portions of the Software.
//
// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
// OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
// SOFTWARE.
//
////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////
//
//
//       ------------
//       INTRODUCTION
//       ------------
//
//   Move/rotate/scale the rectangular object around the window. If it collides with the other rectangular object, it is coloured red, otherwise it is coloured green.
//
//
//       --------
//       CONTROLS
//       --------
//
//   ARROWS			        move
//   < >                    rotate (anti-clockwise/clockwise)
//   [ ]                    scale (shrink/enlarge)
//   SHIFT                  boost speed of move/rotate/scale
//
//
//       -------
//       DEFINES
//       -------
//
//   USE_SPRITES            use sprites (requires USE_TEXTURE)
//   USE_RECTANGLESHAPES    use rectangle shapes
//   USE_TEXTURE            uses texture if defined, uses color only if not defined (must be defined for USE_SPRITES)
//
//   One of either USE_SPRITES or USE_RECTANGLES must be defined but never both
//
//
//       -------
//       TEXTURE
//       -------
//
//   A texture of your choice can be used although one is provided. The path used here is from the root of SfmlSnippets. (SfmlSnippets/resources/images/sw ring rainbow.png)
//
//
////////////////////////////////////////////////////////////////

#include <SFML/Graphics.hpp>

#include "RectangularBoundaryCollision.hpp"

#define USE_TEXTURE
#define USE_SPRITES
//#define USE_RECTANGLESHAPES

int main()
{
    constexpr float movementSpeed{ 150.f };
    constexpr float scaleSpeed{ 1.f };
    constexpr float rotationSpeed{ 75.f };
    constexpr float boostMultiplier{ 3.f };

#ifdef USE_TEXTURE
    sf::Texture texture;
    if (!texture.loadFromFile("resources/images/sw ring rainbow.png"))
        return EXIT_FAILURE;
#endif // USE_TEXTURE

#ifdef USE_RECTANGLESHAPES
    std::vector<sf::RectangleShape> objects(2);
    for (auto& object : objects)
    {
        object.setSize({ 150.f, 100.f });
        object.setOrigin(object.getSize() / 2.f);
#ifdef USE_TEXTURE
        object.setTexture(&texture);
#endif // USE_TEXTURE
    }
    objects[0].setFillColor(sf::Color::Green);
    objects[1].setFillColor(sf::Color(255, 255, 255, 128));
    objects[0].setRotation(10.f);
    objects[1].setRotation(25.f);
#endif // USE_RECTANGLESHAPES
#ifdef USE_SPRITES
    std::vector<sf::Sprite> objects(2, sf::Sprite(texture));
    for (auto& object : objects)
    {
        object.setOrigin(sf::Vector2f(texture.getSize()) / 2.f);
        object.setScale({ 0.5f, 0.5f });
    }
    objects[0].setColor(sf::Color::Green);
    objects[1].setColor(sf::Color(255, 255, 255, 128));
    objects[0].setRotation(10.f);
    objects[1].setRotation(25.f);
#endif // USE_SPRITES

    sf::RenderWindow window(sf::VideoMode(800, 600), "Sprite/Rectangle Collision", sf::Style::Default);
    window.setFramerateLimit(60);

    objects[1].setPosition(sf::Vector2f(window.getSize() / 2u));

    sf::Clock clock;
    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        sf::Time frameTime{ clock.restart() };
        float dt{ frameTime.asSeconds() };

        const float controlMultiplier{ sf::Keyboard::isKeyPressed(sf::Keyboard::LShift) ? boostMultiplier : 1.f };

        sf::Vector2i movementControl;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up))
            --movementControl.y;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down))
            ++movementControl.y;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left))
            --movementControl.x;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right))
            ++movementControl.x;
        sf::Vector2f movement(movementControl);
        if (movementControl.x != 0 && movementControl.y != 0)
            movement *= 0.707f;
        objects[0].move(movement * movementSpeed * controlMultiplier * dt);

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::RBracket))
            objects[0].scale({ 1.f + (scaleSpeed * controlMultiplier * dt), 1.f + (scaleSpeed * controlMultiplier * dt) });
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::LBracket))
            objects[0].scale({ 1.f / (1.f + (scaleSpeed * controlMultiplier * dt)), 1.f / (1.f + (scaleSpeed * controlMultiplier * dt)) });

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Period))
            objects[0].rotate(rotationSpeed * controlMultiplier * dt);
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Comma))
            objects[0].rotate(-rotationSpeed * controlMultiplier * dt);

        const sf::Time collisionStartTime{ clock.getElapsedTime() };
        const bool areColliding{ collision::areColliding(objects[0], objects[1]) }; // this is the collision detection section
        const sf::Time collisionEndTime{ clock.getElapsedTime() };
        const sf::Time collisionDuration{ collisionEndTime - collisionStartTime };

#ifdef USE_RECTANGLESHAPES
        if (areColliding)
            objects[0].setFillColor(sf::Color::Red);
        else
            objects[0].setFillColor(sf::Color::Green);
#endif // USE_RECTANGLESHAPES
#ifdef USE_SPRITES
        if (areColliding)
            objects[0].setColor(sf::Color::Red);
        else
            objects[0].setColor(sf::Color::Green);
#endif // USE_SPRITES

        window.setTitle("collision duration: " + std::to_string(collisionDuration.asMicroseconds()) + "\n");

        window.clear();
        for (auto& object : objects)
            window.draw(object);
        window.display();
    }
}
```

[This example](https://github.com/Hapaxia/SfmlSnippets/blob/master/RectangularBoundaryCollision/example.cpp) is also available on the GitHub repository, along with [the image used](https://github.com/Hapaxia/SfmlSnippets/blob/master/resources/images/sw%20ring%20rainbow.png).

The instructions for this example are contained in comments near the top of the example code.

As visible in the code, the example is licensed using the MIT license.


---
**Written by Hapaxia ([Github](http://github.com/hapaxia) | [Twitter](https://twitter.com/Hapaxiation) | [SFML forum](http://en.sfml-dev.org/forums/index.php?action=profile;u=13086))**