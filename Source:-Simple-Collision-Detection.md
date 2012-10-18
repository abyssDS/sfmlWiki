# Simple Collision Detection
## Overview
Collision Detection is a complex topic with a wide range of algorithms and optimisations, however I've developed a simple static class that implements a few types of collision detection and can provide a starting point for collision detection in peoples projects.

In this code Collision is a static class, I've chosen to do this in order to separate the code from the existing SFML classes.  In your own project you may want to consider subclassing from Sprite and implementing methods in your game objects, or in an object manager.  

## Features & Limitations
Only Sprite to Sprite collision is shown in the code below, the ideas here could be extended to collision detection amongst Shape and String objects as well.

The same Collision method applies to both objects, for example you cannot test a Circle v Box collision.  It is up to your implementation to manage that if necessary.  My game engine has Game Objects extended from sprites that carry their own behaviour information including preferred collision detection method.

Rotated, Scaled and Translated Sprites are supported in all methods including ''PixelPerfectTest''

This code has not been optimised for speed and has a few ugly hacks, but these are commented.  If they can be improved feel free to update the code listing.

## License
This code is under the same license as SFML so can be used in existing projects.

## Methods

The Collision class has three main methods, the choice of the //best// method depends on the shape of the sprite, and the number of tests required per frame:

  * <code>Collision::CircleTest(const sf::Sprite& Object1, const sf::Sprite& Object2)</code>  
Returns true if the two objects are closer together than the sum of their radii.  This is the fastest collision detection method and is excellent for objects that are completely or generally round in shape.  In this code an object's radius is calculated as the average of its width and height, so this test is quite inaccurate for a shape that is much longer than it is wide or vice versa.

  * <code>Collision::BoundingBoxTest(const sf::Sprite& Object1, const sf::Sprite& Object2)</code>  
Returns true if the Oriented Bounding Box (OBB) of the objects collide.  As it is an OBB test a rotated sprite will have diagonal sides and will always be the same size as the sprite, this generally provides a more accurate test than an Axis-Aligned Bounding Box (AABB) where the sides are always aligned with the coordinate system and can create a bigger box than the underlying image.  ''BoundingBoxTest'' is better than ''CircleTest'' for objects that are square or rectangular or are much longer than they are wide.

  * <code>Collision::PixelPerfectTest(const sf::Sprite& Object1, const sf::Sprite& Object2, sf::Uint8 AlphaLimit)</code>  
This hit test is the most accurate, returning true only if the pixels in the sprites are overlapping.  Obviously this is the slowest form of detection, though I have run tests with several hundred objects drawn and tested per frame.  Usually a BoundingBox or Circle test will be sufficient, the PixelPerfectTest should be used for objects that have a very irregular shape, particularly shapes with a significant amount of concavity.  AlphaLimit determines how opaque a pixel needs to be to count as a hit.  A limit of 0 signifies that you want to do a simple AABB test and no pixels will be compared.  A limit of 255 would prevent any pixels being registered as no pixel can have an alpha higher than 255.

## TODO
Using BitMasks is a very quick method for PixelPerfect testing, however most implementations don't easily support scaling and rotating.  If I find a solution for this I may update the PixelPerfect test to use BitMasks.

In order to use these collision test effectively with a large number of sprites you may want to consider organising your sprites into a data structure such as a quad tree in order to reduces the number of tests you need to make.  This is more advanced than the aim of this article.

## Code

To use this code save the following two files and include them into your current SFML project

### Collision.h

```cpp
/* 
 * File:   collision.h
 * Author: Nick Koirala
 *
 * Collision Detection and handling class
 * For SFML.
 
(c) 2009 - LittleMonkey Ltd
 
This software is provided 'as-is', without any express or
implied warranty. In no event will the authors be held
liable for any damages arising from the use of this software.
 
Permission is granted to anyone to use this software for any purpose,
including commercial applications, and to alter it and redistribute
it freely, subject to the following restrictions:
 
1. The origin of this software must not be misrepresented;
   you must not claim that you wrote the original software.
   If you use this software in a product, an acknowledgment
   in the product documentation would be appreciated but
   is not required.
 
2. Altered source versions must be plainly marked as such,
   and must not be misrepresented as being the original software.
 
3. This notice may not be removed or altered from any
   source distribution.
 
 *
 * Created on 30 January 2009, 11:02
 */
 
#ifndef _COLLISION_H
#define	_COLLISION_H
 
 
#ifndef PI
    #define PI (3.14159265358979323846)
#endif
#define RADIANS_PER_DEGREE (PI/180.0)
 
 
class Collision {
public:
 
    virtual ~Collision();
 
    /**
     *  Test for a pixel perfect collision detection between
     *  two Sprites, Rotation and Scale is supported in this test
     *
     *  @param Object1 The first sprite
     *  @param Object2 The second sprite
     *  @AlphaLimit How opaque a pixel needs to be before a hit it registered
     */
     static bool PixelPerfectTest(const sf::Sprite& Object1 ,const sf::Sprite& Object2, sf::Uint8 AlphaLimit = 127);
 
    /**
     *  Test for collision using circle collision dection
     *  Radius is averaged from the dimesnions of the sprite so
     *  roughly circular objects will be much more accurate
     */
    static bool CircleTest(const sf::Sprite& Object1, const sf::Sprite& Object2);
 
    /**
     *  Test for bounding box collision using OBB Box.
     *  To test against AABB use PixelPerfectTest with AlphaLimit = 0
     *
     *  @see Collision::PixelPerfectTest
     */
    static bool BoundingBoxTest(const sf::Sprite& Object1, const sf::Sprite& Object2);
 
    /**
     *  Generate a Axis-Aligned Bounding Box for broad phase of
     *  Pixel Perfect detection.
     *
     *  @returns IntRect to round off Floating point positions.
     */
    static sf::IntRect GetAABB(const sf::Sprite& Object);
 
    /**
     *  Helper function in order to rotate a point by an Angle
     *
     *  Rotation is CounterClockwise in order to match SFML Sprite Rotation
     *
     *  @param Point a Vector2f representing a coordinate
     *  @param Angle angle in degrees
     */
    static sf::Vector2f RotatePoint(const sf::Vector2f& Point, float Angle);
 
    /**
     *  Helper function to get the minimum from a list of values
     */
    static float MinValue(float a, float b, float c, float d);
 
    /**
     *  Helper function to get the maximum from a list of values
     */
    static float MaxValue(float a, float b, float c, float d);
 
private:
 
    Collision();
};
 
#endif	/* _COLLISION_H */
```

### Collision.cpp

```cpp
/* 
 * File:   collision.cpp
 * Author: Nick
 * 
 * Created on 30 January 2009, 11:02
 */
#include <SFML/Graphics.hpp>
#include "Collision.h"
 
Collision::Collision() {
}
 
Collision::~Collision() {
}
 
sf::IntRect Collision::GetAABB(const sf::Sprite& Object) {
 
    //Get the top left corner of the sprite regardless of the sprite's center
    //This is in Global Coordinates so we can put the rectangle back into the right place
    sf::Vector2f pos = Object.TransformToGlobal(sf::Vector2f(0, 0));
 
    //Store the size so we can calculate the other corners
    sf::Vector2f size = Object.GetSize();
 
    float Angle = Object.GetRotation();
 
    //Bail out early if the sprite isn't rotated
    if (Angle == 0.0f) {
        return sf::IntRect(static_cast<int> (pos.x),
                static_cast<int> (pos.y),
                static_cast<int> (pos.x + size.x),
                static_cast<int> (pos.y + size.y));
    }
 
    //Calculate the other points as vectors from (0,0)
    //Imagine sf::Vector2f A(0,0); but its not necessary
    //as rotation is around this point.
    sf::Vector2f B(size.x, 0);
    sf::Vector2f C(size.x, size.y);
    sf::Vector2f D(0, size.y);
 
    //Rotate the points to match the sprite rotation
    B = RotatePoint(B, Angle);
    C = RotatePoint(C, Angle);
    D = RotatePoint(D, Angle);
 
    //Round off to int and set the four corners of our Rect
    int Left = static_cast<int> (MinValue(0.0f, B.x, C.x, D.x));
    int Top = static_cast<int> (MinValue(0.0f, B.y, C.y, D.y));
    int Right = static_cast<int> (MaxValue(0.0f, B.x, C.x, D.x));
    int Bottom = static_cast<int> (MaxValue(0.0f, B.y, C.y, D.y));
 
    //Create a Rect from out points and move it back to the correct position on the screen
    sf::IntRect AABB = sf::IntRect(Left, Top, Right, Bottom);
    AABB.Offset(static_cast<int> (pos.x), static_cast<int> (pos.y));
    return AABB;
}
 
float Collision::MinValue(float a, float b, float c, float d) {
    float min = a;
 
    min = (b < min ? b : min);
    min = (c < min ? c : min);
    min = (d < min ? d : min);
 
    return min;
}
 
float Collision::MaxValue(float a, float b, float c, float d) {
    float max = a;
 
    max = (b > max ? b : max);
    max = (c > max ? c : max);
    max = (d > max ? d : max);
 
    return max;
}
 
sf::Vector2f Collision::RotatePoint(const sf::Vector2f& Point, float Angle) {
    Angle = Angle * RADIANS_PER_DEGREE;
    sf::Vector2f RotatedPoint;
    RotatedPoint.x = Point.x * cos(Angle) + Point.y * sin(Angle);
    RotatedPoint.y = -Point.x * sin(Angle) + Point.y * cos(Angle);
    return RotatedPoint;
}
 
bool Collision::PixelPerfectTest(const sf::Sprite& Object1, const sf::Sprite& Object2, sf::Uint8 AlphaLimit) {
        //Get AABBs of the two sprites
    sf::IntRect Object1AABB = GetAABB(Object1);
    sf::IntRect Object2AABB = GetAABB(Object2);
 
    sf::IntRect Intersection;
 
    if (Object1AABB.Intersects(Object2AABB, &Intersection)) {
 
        //We've got an intersection we need to process the pixels
        //In that Rect.
 
        //Bail out now if AlphaLimit = 0
        if (AlphaLimit == 0) return true;
 
        //There are a few hacks here, sometimes the TransformToLocal returns negative points
        //Or Points outside the image.  We need to check for these as they print to the error console
        //which is slow, and then return black which registers as a hit.
 
        sf::IntRect O1SubRect = Object1.GetSubRect();
        sf::IntRect O2SubRect = Object2.GetSubRect();
 
        sf::Vector2i O1SubRectSize(O1SubRect.GetWidth(), O1SubRect.GetHeight());
        sf::Vector2i O2SubRectSize(O2SubRect.GetWidth(), O2SubRect.GetHeight());
 
        sf::Vector2f o1v;
        sf::Vector2f o2v;
        //Loop through our pixels
        for (int i = Intersection.Left; i < Intersection.Right; i++) {
            for (int j = Intersection.Top; j < Intersection.Bottom; j++) {
 
                o1v = Object1.TransformToLocal(sf::Vector2f(i, j)); //Creating Objects each loop :(
                o2v = Object2.TransformToLocal(sf::Vector2f(i, j));
 
                //Hack to make sure pixels fall withint the Sprite's Image
                if (o1v.x > 0 && o1v.y > 0 && o2v.x > 0 && o2v.y > 0 &&
                        o1v.x < O1SubRectSize.x && o1v.y < O1SubRectSize.y &&
                        o2v.x < O2SubRectSize.x && o2v.y < O2SubRectSize.y) {
 
                    //If both sprites have opaque pixels at the same point we've got a hit
                    if ((Object1.GetPixel(static_cast<int> (o1v.x), static_cast<int> (o1v.y)).a > AlphaLimit) &&
                            (Object2.GetPixel(static_cast<int> (o2v.x), static_cast<int> (o2v.y)).a > AlphaLimit)) {
                        return true;
                    }
                }
            }
        }
        return false;
    }
    return false;
}
 
bool Collision::CircleTest(const sf::Sprite& Object1, const sf::Sprite& Object2) {
    //Simplest circle test possible
    //Distance between points <= sum of radius
 
    float Radius1 = (Object1.GetSize().x + Object1.GetSize().y) / 4;
    float Radius2 = (Object2.GetSize().x + Object2.GetSize().y) / 4;
    float xd = Object1.GetPosition().x - Object2.GetPosition().x;
    float yd = Object1.GetPosition().y - Object2.GetPosition().y;
 
    return sqrt(xd * xd + yd * yd) <= Radius1 + Radius2;
}
 
//From Rotated Rectangles Collision Detection, Oren Becker, 2001
 
bool Collision::BoundingBoxTest(const sf::Sprite& Object1, const sf::Sprite& Object2) {
 
    sf::Vector2f A, B, C, BL, TR;
    sf::Vector2f HalfSize1 = Object1.GetSize();
    sf::Vector2f HalfSize2 = Object2.GetSize();
 
    //For somereason the Vector2d divide by operator
    //was misbehaving
    //Doing it manually
    HalfSize1.x /= 2;
    HalfSize1.y /= 2;
    HalfSize2.x /= 2;
    HalfSize2.y /= 2;
    //Get the Angle we're working on
    float Angle = Object1.GetRotation() - Object2.GetRotation();
    float CosA = cos(Angle * RADIANS_PER_DEGREE);
    float SinA = sin(Angle * RADIANS_PER_DEGREE);
 
    float t, x, a, dx, ext1, ext2;
 
    //Normalise the Center of Object2 so its axis aligned an represented in
    //relation to Object 1
    C = Object2.GetPosition();
 
    C -= Object1.GetPosition();
 
    C = RotatePoint(C, Object2.GetRotation());
 
    //Get the Corners
    BL = TR = C;
    BL -= HalfSize2;
    TR += HalfSize2;
 
    //Calculate the vertices of the rotate Rect
    A.x = -HalfSize1.y*SinA;
    B.x = A.x;
    t = HalfSize1.x*CosA;
    A.x += t;
    B.x -= t;
 
    A.y = HalfSize1.y*CosA;
    B.y = A.y;
    t = HalfSize1.x*SinA;
    A.y += t;
    B.y -= t;
 
    t = SinA * CosA;
 
    // verify that A is vertical min/max, B is horizontal min/max
    if (t < 0) {
        t = A.x;
        A.x = B.x;
        B.x = t;
        t = A.y;
        A.y = B.y;
        B.y = t;
    }
 
    // verify that B is horizontal minimum (leftest-vertex)
    if (SinA < 0) {
        B.x = -B.x;
        B.y = -B.y;
    }
 
    // if rr2(ma) isn't in the horizontal range of
    // colliding with rr1(r), collision is impossible
    if (B.x > TR.x || B.x > -BL.x) return false;
 
    // if rr1(r) is axis-aligned, vertical min/max are easy to get
    if (t == 0) {
        ext1 = A.y;
        ext2 = -ext1;
    }// else, find vertical min/max in the range [BL.x, TR.x]
    else {
        x = BL.x - A.x;
        a = TR.x - A.x;
        ext1 = A.y;
        // if the first vertical min/max isn't in (BL.x, TR.x), then
        // find the vertical min/max on BL.x or on TR.x
        if (a * x > 0) {
            dx = A.x;
            if (x < 0) {
                dx -= B.x;
                ext1 -= B.y;
                x = a;
            } else {
                dx += B.x;
                ext1 += B.y;
            }
            ext1 *= x;
            ext1 /= dx;
            ext1 += A.y;
        }
 
        x = BL.x + A.x;
        a = TR.x + A.x;
        ext2 = -A.y;
        // if the second vertical min/max isn't in (BL.x, TR.x), then
        // find the local vertical min/max on BL.x or on TR.x
        if (a * x > 0) {
            dx = -A.x;
            if (x < 0) {
                dx -= B.x;
                ext2 -= B.y;
                x = a;
            } else {
                dx += B.x;
                ext2 += B.y;
            }
            ext2 *= x;
            ext2 /= dx;
            ext2 -= A.y;
        }
    }
 
    // check whether rr2(ma) is in the vertical range of colliding with rr1(r)
    // (for the horizontal range of rr2)
    return !((ext1 < BL.y && ext2 < BL.y) ||
            (ext1 > TR.y && ext2 > TR.y));
 
}
```