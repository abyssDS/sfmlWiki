This page as its name describes is a high performance container of sprites, if you have a low end computer you may have found that a vector of sprites with too many sprites in it can have a slow performance due to the high amount of draw calls. 

The only way to avoid the just described problem is to use an sf::VertexArray or a std::vector of sf::Vertex which is almost the same thing, the problem with this is that it becomes somewhat complicated to program and eventually you are going to need a wrapper that works things out and makes your code cleaner.

That's the purpose of this container, while each transformation is a little slower than a transformation powered by sf:.Transform it's still faster than a std::vector of sf::Sprites by cutting the amount of time spent drawing in half, and at worst (if you have a bad and old GPU like my intel crap) it will have the same performance not because the container is slow, but because the display function will screw your framerate at a high number of things being displayed.

This container is useful not only for increasing your performances (even if just slightly in some cases), but also as a shortcut, it has shortcut functions such as all "global" transformations that affect the entire container with the values given to the function. Something also worth mentioning is the function "rotateAroundSelf" which is a shortcut that allows your "sprites" to rotate around their center.

As a side-note: This code uses my polar vector class to power rotations, I will include the code of it here, but you can perfectly use Thor's polar vectors or even your own, it's not that hard to modify.

Without further ado I present the code:

```cpp
///Polar Vector class
#include <SFML/System/Vector2.hpp>

#pragma once

///The class uses radians as angles, where r is the radius and t is the angle.

class PolarVector
{ 
  public:
        float r;
        float t;

        PolarVector();
        PolarVector(float radius, float angle);

        sf::Vector2f TurnToRectangular() const;
};

PolarVector TurnToPolar(const sf::Vector2f& point);

bool operator ==(const PolarVector& left, const PolarVector& right);

bool operator !=(const PolarVector& left, const PolarVector& right);

///.cpp

#include "PolarVector.hpp"
#include <cmath>

#define EPSILON 0.0001

PolarVector::PolarVector()
  :r(0.0), t(0.0)
{}

PolarVector::PolarVector(float radius, float angle)
  :r(radius), t(angle)
{}

sf::Vector2f PolarVector::TurnToRectangular() const
{
  sf::Vector2f Rect;
  Rect.x = static_cast<float>(r*std::cos(t)); Rect.y = static_cast<float>(r*std::sin(t));
  return Rect;
}

bool operator ==(const PolarVector& left, const PolarVector& right)
{
  float diffR = left.r - right.r;
  float diffA = left.t - right.t;
  return ((diffR <= EPSILON) && (diffA <= EPSILON));
}

bool operator !=(const PolarVector& left, const PolarVector& right)
{
  float diffR = left.r - right.r;
  float diffA = left.t - right.t;
  return !((diffR <= EPSILON) && (diffA <= EPSILON));
}

PolarVector TurnToPolar(const sf::Vector2f& point)
{
    PolarVector PV;
    PV.r = sqrt((point.x * point.x) + (point.y * point.y));
    PV.t = std::atan2(point.y, point.x);
    return PV;
}

///Sprite Container class
///.hpp

#ifndef ALTSPRITEHOLDER_HPP
#define ALTSPRITEHOLDER_HPP

#include <vector>

#include <SFML/Graphics/Drawable.hpp>
#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Transform.hpp>

#include <SFML/Graphics/Vertex.hpp>
#include <SFML/Graphics/Texture.hpp>

const float APi = 3.141592654;
const float toRadians = APi/180;
const float toDegrees = 180/APi;


class AltSpriteHolder : public sf::Drawable
{
    public:
        AltSpriteHolder(const unsigned amount);
        AltSpriteHolder(sf::Texture& T, const unsigned amount);
        ~AltSpriteHolder();

        AltSpriteHolder(const AltSpriteHolder& other);
        AltSpriteHolder& operator=(const AltSpriteHolder& other);

        void setTexture(sf::Texture& T);

        void setTextureRect(const unsigned index, const sf::IntRect& IR);
        void setGlobalTextureRect(const sf::IntRect& IR);

        void move(const unsigned index, const sf::Vector2f& V);
        void move(const unsigned index, const float x, const float y);
        void globalMove(const sf::Vector2f& V);

        void setPosition(const unsigned index, const sf::Vector2f& V);
        void setPosition(const unsigned index, const float x, const float y);
        void setGlobalPosition(const sf::Vector2f& V);

        void setRotation(const unsigned index, const float ang);
        void setGlobalRotation(const float ang);

        void rotate(const unsigned index, const float ang);
        void globalRotate(const float ang);

        void rotateAroundSelf(const unsigned index, const float ang);
        void globalRotateAroundSelf(const float ang);

        void resetSelfRotation(const unsigned index);
        void resetGlobalSelfRotation();

        void scale(const unsigned index, const sf::Vector2f& V);
        void globalScale(const sf::Vector2f& V);

        void setScale(const unsigned index, const float x, const float y);
        void setScale(const unsigned index, const sf::Vector2f& V);
        void setGlobalScale(const sf::Vector2f& V);

        sf::Vector2f getPosition(const unsigned index) const;
        sf::Vector2f getScale(const unsigned index) const;
        float getRotation(const unsigned index) const;

        void draw(sf::RenderTarget& target, sf::RenderStates states) const;

    private:

        unsigned Quantity;

        sf::Texture* Tex;

        std::vector<sf::Vertex> VertexHolder;

        std::vector<float> AngleHolder;
        std::vector<sf::Vector2f> PositionHolder;
        std::vector<sf::Vector2f> ScaleHolder;
        std::vector<sf::IntRect> TexRectHolder;

        bool hasTexture;

        void updateTexCoords(const unsigned index);
        void updateVertexCoords(const unsigned index, const bool Reset = true);

        void updateAngleRanges(const unsigned index);

};

#endif // ALTSPRITEHOLDER_HPP

/// .cpp

#include "AltSpriteHolder.hpp"
#include "PolarVector.hpp"
#include <cmath>

AltSpriteHolder::AltSpriteHolder(const unsigned amount)
  :VertexHolder(), PositionHolder(), Quantity(amount), ScaleHolder(), AngleHolder(), Tex(nullptr)
{
    VertexHolder.reserve(amount*4);
    PositionHolder.reserve(amount);
    ScaleHolder.reserve(amount);
    AngleHolder.reserve(amount);

    for(unsigned i = 0; i < amount; ++i)
    {
        for(unsigned j = 0; j < 4; ++j)
        { VertexHolder.push_back(sf::Vertex(sf::Vector2f(0.f, 0.f), sf::Color(255, 255, 255, 255))); }

        PositionHolder.push_back(sf::Vector2f(0, 0));
        ScaleHolder.push_back(sf::Vector2f(1.f, 1.f));
        AngleHolder.push_back(0.f);
        TexRectHolder.push_back(sf::IntRect(0, 0, 0, 0));
    }
}

AltSpriteHolder::AltSpriteHolder(sf::Texture& T, const unsigned amount)
  :VertexHolder(), PositionHolder(), Quantity(amount), ScaleHolder(), AngleHolder(), Tex(&T)
{
    VertexHolder.reserve(amount * 4);
    PositionHolder.reserve(amount);
    ScaleHolder.reserve(amount);
    AngleHolder.reserve(amount);

    for(unsigned i = 0; i < amount; ++i)
    {
        for(unsigned j = 0; j < 4; ++j)
        { VertexHolder.push_back(sf::Vertex(sf::Vector2f(0.f, 0.f), sf::Color(255, 255, 255, 255))); }

        PositionHolder.push_back(sf::Vector2f(0, 0));
        ScaleHolder.push_back(sf::Vector2f(1.f, 1.f));
        AngleHolder.push_back(0.f);
        setTextureRect(i, sf::IntRect(0, 0, Tex->getSize().x, Tex->getSize().y));
    }
}

AltSpriteHolder::~AltSpriteHolder()
{
    VertexHolder.clear();
    PositionHolder.clear();
    ScaleHolder.clear();
    AngleHolder.clear();
    TexRectHolder.clear();
}

AltSpriteHolder::AltSpriteHolder(const AltSpriteHolder& other)
{
    //copy ctor
}

AltSpriteHolder& AltSpriteHolder::operator=(const AltSpriteHolder& rhs)
{
    if (this == &rhs) return *this; // handle self assignment
    //assignment operator
    return *this;
}

void AltSpriteHolder::draw(sf::RenderTarget& target, sf::RenderStates states) const
{
    states.texture = Tex;
    target.draw(&VertexHolder[0], static_cast<unsigned>(VertexHolder.size()), sf::Quads, states);
}

void AltSpriteHolder::setTexture(sf::Texture& T)
{
    Tex = &T;
    setGlobalTextureRect(sf::IntRect(0, 0, Tex->getSize().x, Tex->getSize().y) );
}

void AltSpriteHolder::setTextureRect(const unsigned index, const sf::IntRect& IR)
{
    TexRectHolder[index] = IR;
    updateTexCoords(index);
    updateVertexCoords(index, false);
}

void AltSpriteHolder::setGlobalTextureRect(const sf::IntRect& IR)
{
    for(unsigned i = 0; i < Quantity; ++i)
    { setTextureRect(i, IR); }
}

void AltSpriteHolder::updateTexCoords(const unsigned index)
{
    float left = static_cast<float>(TexRectHolder[index].left);
    float right = left + TexRectHolder[index].width;
    float top = static_cast<float>(TexRectHolder[index].top);
    float bottom = top + TexRectHolder[index].height;

    unsigned I = index * 4;

    VertexHolder[I].texCoords = sf::Vector2f(left, top);
    ++I;
    VertexHolder[I].texCoords = sf::Vector2f(left, bottom);
    ++I;
    VertexHolder[I].texCoords = sf::Vector2f(right, bottom);
    ++I;
    VertexHolder[I].texCoords = sf::Vector2f(right, top);
}

void AltSpriteHolder::updateVertexCoords(const unsigned index, const bool Reset)
{
    sf::Vector2u S;
    S.x = TexRectHolder[index].width * ScaleHolder[index].x;
    S.y = TexRectHolder[index].height * ScaleHolder[index].y;

    unsigned I = index * 4;

    VertexHolder[I].position = sf::Vector2f(0, 0);
    ++I;
    VertexHolder[I].position = sf::Vector2f(0, S.y);
    ++I;
    VertexHolder[I].position = sf::Vector2f(S.x, S.y);
    ++I;
    VertexHolder[I].position = sf::Vector2f(S.x, 0);

    if(Reset)
    {
    PositionHolder[index].x = 0.f;
    PositionHolder[index].y = 0.f;
    }
    else
    { move(index, PositionHolder[index]); }
}

void AltSpriteHolder::move(const unsigned index, const float x, const float y)
{
    unsigned limit = (index * 4) + 4;
    for(unsigned I = index * 4; I < limit; ++I)
    {
        VertexHolder[I].position.x += x;
        VertexHolder[I].position.y += y;
    }
    PositionHolder[index].x += x;
    PositionHolder[index].y += y;
}

void AltSpriteHolder::move(const unsigned index, const sf::Vector2f& V)
{ move(index, V.x, V.y); }

void AltSpriteHolder::globalMove(const sf::Vector2f& V)
{
    for(unsigned i = 0; i < Quantity; ++i)
    { move(i, V.x, V.y); }
}

void AltSpriteHolder::setPosition(const unsigned index, const float x, const float y)
{
    move(index, -(PositionHolder[index].x), -(PositionHolder[index].y));
    move(index, x, y);
}

void AltSpriteHolder::setPosition(const unsigned index, const sf::Vector2f& V)
{ setPosition(index, V.x, V.y); }

void AltSpriteHolder::setGlobalPosition(const sf::Vector2f& V)
{
    for(unsigned i = 0; i < Quantity; ++i)
    { setPosition(i, V); }
}

void AltSpriteHolder::setRotation(const unsigned index, const float ang)
{
    float ang_t = ang;
    AngleHolder[index] = fmod(ang, 360.f);
    PolarVector P = TurnToPolar(PositionHolder[index]); P.t = ang_t;
    setPosition(index, P.TurnToRectangular() );
}

void AltSpriteHolder::setGlobalRotation(const float ang)
{
    for(unsigned i = 0; i < Quantity; ++i)
    { setRotation(i, ang); }
}

void AltSpriteHolder::rotate(const unsigned index, const float ang)
{ setRotation(index, AngleHolder[index] + ang); }

void AltSpriteHolder::globalRotate(const float ang)
{
    for(unsigned i = 0; i < Quantity; ++i)
    { rotate(i, ang); }
}

void AltSpriteHolder::rotateAroundSelf(const unsigned index, const float ang)
{
    float ang_t = ang * toRadians;
    sf::Vector2f Tmp = PositionHolder[index];
    sf::Vector2f Off(-(TexRectHolder[index].width/2.f), -(TexRectHolder[index].height/2.f));

    setPosition(index, Off);
    unsigned limit = (index * 4) + 4;
    for(unsigned i = index * 4; i < limit; ++i)
    {
        PolarVector P(TurnToPolar(VertexHolder[i].position));
        P.t += ang_t;
        VertexHolder[i].position = P.TurnToRectangular();
    }
    move(index, Tmp - Off);
}

void AltSpriteHolder::globalRotateAroundSelf(const float ang)
{
    for(unsigned i = 0; i < Quantity; ++i)
    { rotateAroundSelf(i, ang); }
}

void AltSpriteHolder::resetSelfRotation(const unsigned index)
{ updateVertexCoords(index, false); }

void AltSpriteHolder::resetGlobalSelfRotation()
{
    for(unsigned i = 0; i < Quantity; ++i)
    { updateVertexCoords(i, false); }
}

void AltSpriteHolder::setScale(const unsigned index, const float x, const float y)
{
    ScaleHolder[index].x = x;
    ScaleHolder[index].y = y;
    for(unsigned i = index * 4; i < (index * 4) + 4; ++i)
    {
        VertexHolder[i].position.x *= x;
        VertexHolder[i].position.y *= y;
    }
}

void AltSpriteHolder::setScale(const unsigned index, const sf::Vector2f& V)
{ setScale(index, V.x, V.y); }

void AltSpriteHolder::setGlobalScale(const sf::Vector2f& V)
{
    for(unsigned i = 0; i < Quantity; ++i)
    { setScale(i, V); }
}

sf::Vector2f AltSpriteHolder::getPosition(const unsigned index) const
{ return PositionHolder[index]; }

sf::Vector2f AltSpriteHolder::getScale(const unsigned index) const
{ return ScaleHolder[index]; }

float AltSpriteHolder::getRotation(const unsigned index) const
{ return AngleHolder[index]; }

```

If you have any doubts or bugs to report PM me in the SFML forums (masskiller).

