# Introduction
SFML's sprites and shapes can provide you with their axis-aligned boundary boxes (getGlobalBounds) and you can easily use these rectangles to test if they intersect. Unfortunately, this _only_ allows axis-aligned collision detection.

Rectangular Boundary Collision allows testing collision based on their original rectangular boundary box (getLocalBounds).
It tests to see if and object's rectangular boundary collides with another object's rectangular boundary. This method, however, allows transformations to be taken into account - most notably, rotation.

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

If you would prefer to have the code directly included in another file (header or source), you can remove the include guards (lines starting with #ifdef, #define and #endif)

## Function
The function to use is "areColliding" and it returns a boolean that represents if the two objects are colliding.

```cpp
template <class T1, class T2>
bool areColliding(const T1& object1, const T2& object2, const int collisionLevel = -1);
```

The two objects should be provided as (the first two) parameters to the function; the two object types can be different.
> Note that any objects can be used as long as they have these four functions: getLocalBounds (must return sf::FloatRect), getTransform and getInverseTransform (both must return sf::Transform)

The third parameter is optional. It specifies the required "collision level".

## Collision Level
The collision is performed in three stages:

- Level 0 - AABB (axis-aligned boundaries boxes):  
tests the axis-aligned boundaries boxes (equivalent to getGlobalBounds) of the objects to see if they intersect.

- Level 1 - Corner inside opposite object:  
tests to see if any of either object's corner is inside the other object.

- Level 2 - SAT (Separating Axis Theorem):  
tests using a simplified version (for rectangles) of SAT to catch other cases.

If, after any stage, the result is certain, the following stages are not calculated. This allows easier detections to be done without resorting to the more involved calculations.

The collision level represents after which stage to leave the function, regardless of if a result is certain.

If a negative value is provided for Collision Level (its default value), the maximum level (2) is used.

# Namespace
The function(s) in this code are contained within the "collision" namespace.

To call the function, you can use something like:
`const bool spritesAreColliding = collision::areColliding(sprite1, sprite2);`

# License
This code is from https://github.com/Hapaxia/SfmlSnippets/tree/master/RectangularBoundaryCollision and [provided under the zlib license](https://github.com/Hapaxia/SfmlSnippets/blob/master/RectangularBoundaryCollision/LICENSE.txt).

# Example
[An example is also available on the GitHub Repository](https://github.com/Hapaxia/SfmlSnippets/blob/master/RectangularBoundaryCollision/example.cpp).

As visible in the code, the example is licensed using the MIT license.


---
**Written by Hapaxia ([Github](http://github.com/hapaxia) | [Twitter](https://twitter.com/Hapaxiation) | [SFML forum](http://en.sfml-dev.org/forums/index.php?action=profile;u=13086))**