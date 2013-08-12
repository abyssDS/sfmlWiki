# Verlet physic
<a name="top" />

This code provide simple class to simulate circle body in space.
All body are attracted by all other body.
this code work fine with SFML 2.0 ( tested on Linux kubuntu x64 )

![verlet physic balls](http://img15.hostingpics.net/pics/491696capturedcran1.png)

```cpp
#include <math.h>
#include <cstdlib>
#include <vector>
#include <iostream>
#include <time.h>

#include <SFML/Graphics.hpp>
#include <SFML/Window.hpp>

#define GENERATE_BODY  512

class Verlet
{
public:
    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    Verlet()
    {
        _position.x   = 0;
        _position.y   = 0;
        _lastPosition = _position;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    Verlet(const sf::Vector2f & pos)
    {
        _position               = pos;
        _lastPosition   = pos;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    void Update(float timeStep)
    {
        sf::Vector2f temp = _position;
        _position                += _position - _lastPosition + _acceleration * timeStep * timeStep  ;
        _lastPosition     = temp;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    sf::Vector2f & GetPosition()
    {
        return _position;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    void SetAcceleration(const sf::Vector2f & accel)
    {
        _acceleration = accel;
    }

protected:
    sf::Vector2f            _position;
    sf::Vector2f            _lastPosition;
    sf::Vector2f            _acceleration;

};

class CircleBody;

typedef struct CollideInfo CollideInfo;
struct CollideInfo
{
    float                   depth;
    sf::Vector2f    normal;
    CircleBody *    body;
};

class CircleBody :  public Verlet,
    public sf::CircleShape
{
public:
    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    CircleBody()
    {
        Verlet::_position.x     = 0.0f;
        Verlet::_position.y     = 0.0f;
        Verlet::_lastPosition.x = 0.0f;
        Verlet::_lastPosition.y = 0.0f;
        Verlet::_acceleration.x = 0.0f;
        Verlet::_acceleration.y = 0.0f;
        _updatable = false;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    CircleBody(const sf::Vector2f & pos, float radius, float state)
    {
        _position               = pos;
        _lastPosition   = pos;
        _acceleration   = sf::Vector2f(0,0);
        _radius                 = radius;
        _mass                   = _radius * _radius;
        _updatable              = state;

        sf::CircleShape::setRadius(_radius);
        sf::CircleShape::setPosition(_position);
        sf::CircleShape::setFillColor(sf::Color(64,128,255,150));
        sf::CircleShape::setOutlineColor(sf::Color(255,255,255,128));
        sf::CircleShape::setOutlineThickness(2.0f);

    }
    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    float GetRadius()
    {
        return _radius;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    void Update(float timeStep)
    {

        if(_updatable == true)
        {
            Verlet::Update(timeStep);
        }

        sf::CircleShape::setPosition(sf::Vector2f(_position.x-_radius ,
                                     _position.y-_radius));
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    bool IsUpdatable()
    {
        return _updatable;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    float GetMass()
    {
        return _mass;
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    bool CheckCollide(CircleBody & other)
    {
        if (&other != this)
        {
            sf::Vector2f temp;
            temp = other.Verlet::GetPosition() - _position;
            float dist = sqrt(temp.x * temp.x + temp.y * temp.y);

            if(dist >= _radius + other.GetRadius())
            {
                return false;
            }

            if( dist > 1e-08)
            {
                temp.x  /= dist;
                temp.y  /= dist;
            }

            _collide.normal = temp;
            _collide.depth  = _radius + other.GetRadius() - dist;
            _collide.body   = &other;

            if(_collide.depth<1e-08)
                _collide.depth=1e-08;

            return true;
        }
    }

    ///////////////////////////////////////////////////////////////////////
    //
    //
    ///////////////////////////////////////////////////////////////////////
    void Collide(CircleBody & other)
    {
        if (_updatable == true)
        {
            if(other.IsUpdatable() == true)
            {
                sf::Vector2f move;
                move = _collide.normal * _collide.depth * 0.5f;

                _position -= move;
                other.Verlet::GetPosition() += move;
            }
            else
            {
                sf::Vector2f move;
                move = _collide.normal * _collide.depth;
                _position -= move;
            }
        }
    }

protected:

    float           _mass;
    float           _radius;
    bool            _updatable;
    CollideInfo     _collide;
};

int main()
{
    sf::VideoMode desktop = sf::VideoMode::getDesktopMode();
    sf::RenderWindow window(desktop, "SFML window", sf::Style::Fullscreen);
    window.setFramerateLimit(60);

    sf::Font font;
    if (!font.loadFromFile("Arial.ttf"))
        return EXIT_FAILURE;

    sf::Text text("SIMPLE BODY TEST BY CPL.BATOR\nARROW KEY TO SCROLL\nPAGE UP/DOWN TO ZOOM", font, 240);
    text.setStyle(sf::Text::Bold);
    text.setColor(sf::Color(255,128,64,128));
    text.scale(0.25f,0.25f);

    float zoomFactor = 1.0f;
    bool zoomUp      = false;
    bool zoomDown    = false;
    sf::View view;
    view = window.getView();

    std::vector<CircleBody*>       body;

    srand ( time(NULL) );

    for (int i=0; i < GENERATE_BODY / 2; ++i)
    {

        int px = -(rand() % 500 + 1) + (rand() % 1500 + 1);
        int py = -(rand() % 500 + 1) + (rand() % 1500 + 1);

        CircleBody * b = new CircleBody(sf::Vector2f((float)px,(float)py), rand() % 25 + 1, true);
        body.push_back(b);
    }

    float                  ghost_radius = 15.0f;
    sf::Vector2i           ghost_position;
    sf::CircleShape        ghost;

    ghost.setRadius(ghost_radius);
    ghost.setFillColor(sf::Color(255,255,255,64));

    while (window.isOpen())
    {

        sf::Event event;
        while (window.pollEvent(event))
        {

            if (event.type == sf::Event::Closed)
                window.close();

            if(event.type == sf::Event::MouseWheelMoved)
            {
                if(event.mouseWheel.delta > 0)
                {
                    ghost_radius += 1;
                }
                else if(event.mouseWheel.delta < 0)
                {
                    ghost_radius -= 1;
                }
            }

            if(event.type == sf::Event::MouseButtonReleased)
            {
                if (event.mouseButton.button == sf::Mouse::Left)
                {
                    sf::Vector2f   pos = window.mapPixelToCoords(sf::Vector2i(ghost_position.x - ghost_radius ,ghost_position.y - ghost_radius));

                    CircleBody * b = new CircleBody(pos, ghost_radius , true);
                    body.push_back(b);
                }
            }

        }

        ghost.setRadius(ghost_radius);

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left))
        {
            view.move(-10.0f,0.0f);
        }
        else if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right))
        {
            view.move(10.0f,0.0f);
        }
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up))
        {
            view.move(0.0f,-10.0f);
        }
        else if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down))
        {
            view.move(0.0f,10.0f);
        }
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::PageUp))
        {
            zoomFactor += 0.01;
            zoomUp          = true;
            zoomDown        = false;
        }
        else if (sf::Keyboard::isKeyPressed(sf::Keyboard::PageDown))
        {
            zoomFactor -= 0.01;
            zoomUp          = false;
            zoomDown        = true;
        }


        if ( zoomUp == true && zoomFactor > 1.0f)
        {
            zoomFactor -= 0.005f;
            if (zoomFactor <= 1.0 )
            {
                zoomFactor = 1.0f;
                zoomUp = false;
            }
        }

        if ( zoomDown == true && zoomFactor < 1.0f)
        {
            zoomFactor += 0.005f;
            if (zoomFactor >= 1.0 )
            {
                zoomFactor = 1.0f;
                zoomDown = false;
            }
        }

        if (zoomFactor > 1.2f)
            zoomFactor = 1.2f;

        if (zoomFactor < 0.8f)
            zoomFactor = 0.8f;

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Escape))
        {
            window.close();
        }

        std::vector<CircleBody*>::iterator itA;
        std::vector<CircleBody*>::iterator itB;

        for(itA = body.begin(); itA<body.end(); ++itA)
        {

            CircleBody * bodyA = *itA;
            sf::Vector2f accel(0.0f,0.0f);
            // Update position
            for(itB = body.begin(); itB<body.end(); ++itB)
            {

                CircleBody * bodyB = *itB;

                if(*itA != *itB)
                {
                    sf::Vector2f temp = bodyB->Verlet::GetPosition() - bodyA->Verlet::GetPosition();

                    float dist = sqrt(temp.x * temp.x + temp.y * temp.y);
                    if(dist>1e-08)
                    {
                        temp.x /= dist;
                        temp.y /= dist;
                    }

                    dist = bodyA->GetMass() * bodyB->GetMass() * 0.001f / (dist * dist);

                    temp *= dist;

                    accel += temp;
                }
            }

            bodyA->Verlet::SetAcceleration(accel);

            // Check for collision
            for(itB = body.begin(); itB<body.end(); ++itB)
            {
                CircleBody * bodyB = *itB;

                if(*itA != *itB)
                {
                    if (bodyA->CheckCollide(*bodyB) == true)
                    {
                        bodyA->Collide(*bodyB);
                    }
                }
            }

            bodyA->Update(2);

        }

        view.zoom(zoomFactor);
        window.setView(view);

        ghost_position   =  sf::Mouse::getPosition(window);
        sf::Vector2f   v = window.mapPixelToCoords(sf::Vector2i( ghost_position.x - ghost_radius ,ghost_position.y - ghost_radius));

        ghost.setPosition(v.x, v.y);

        window.clear(sf::Color(2,4,8,255));

        window.draw(text);

        std::vector<CircleBody*>::iterator it;
        for(it = body.begin(); it<body.end(); ++it)
        {
            window.draw(**it);
        }

        window.draw(ghost);

        window.display();
    }

    return 1;
}
```
