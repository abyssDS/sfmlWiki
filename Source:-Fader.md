# Fader
## Overview
A class that facilitates fading out or fading in the drawing area of an sf::RenderWindow, while still allowing the drawing area to be dynamically updated.  There is a bit of C++11 sprinkled about in the code.  Notably std::function/std::bind.  If you're using an older compiler you can probably bring those in via TR1 or use the boost equivalent.

Feel free to use the code for whatever you care to.
## Source

### Fader.h
    #ifndef FADER_H_
    #define FADER_H_

    #include <functional>
    #include <SFML/Graphics/RenderWindow.hpp>
    #include <SFML/Graphics/RectangleShape.hpp>
    #include <SFML/Window/Event.hpp>
    #include <SFML/System/Time.hpp>

    class Fader
    {
    public:
        typedef std::function<void(sf::Event&)> onEventUpdate ;
        typedef std::function<void()> onDrawUpdate ;
        enum Dir { Out, In } ;

   
        const static sf::Time defaultTimestep ;

        Fader(sf::RenderWindow& window,                      // The window to manipulate. 
                onDrawUpdate drawUpdate,                     // The callback for drawing updates
                Dir direction = Out,                         // Fading in or fading out?
                onEventUpdate eventUpdate = nullptr,         // The callback for events
                sf::Color faded=sf::Color(0,0,0),            // The color when fully faded.
                sf::Time fadeDuration= sf::seconds(1.0),     // The duration to fully fade/unfade
                sf::Time timeStep = defaultTimestep) ;       // The target timestep for the drawUpdate callback.

    private:

        void _workIt() ;

        sf::RenderWindow & _window ;

        onEventUpdate _eventUpdate ;
        onDrawUpdate _drawUpdate ;

        Dir _direction ;

        sf::Color _faded ;

        sf::RectangleShape _overlay ;

        sf::Time _timestep ;
        sf::Time _fadeDuration ;

        Fader(const Fader &) ;
        Fader& operator=(const Fader&);
    };

    #endif

### Fader.cpp
    #include "Fader.h"

    namespace{  // utility functions:
        template <typename T>
        sf::Vector2f to_v2f(T v)
            { return sf::Vector2f(static_cast<float>(v.x), static_cast<float>(v.y)) ; }

        unsigned clamp(float value)
        {
            if ( value < 0.1 )
                return 0 ;
            if ( value > 254.9 )
                return 255 ;
            return static_cast<unsigned>(value) ;
        }
    }


    const sf::Time Fader::defaultTimestep = sf::seconds(1.0f / 60.0f) ;

    Fader::Fader(sf::RenderWindow& window, onDrawUpdate drawUpdate, Dir direction, onEventUpdate eventUpdate, 
                 sf::Color faded, sf::Time fadeDuration, sf::Time timeStep)
        : _window(window), _eventUpdate(eventUpdate), _drawUpdate(drawUpdate), _direction(direction), _faded(faded), 
          _overlay(to_v2f(_window.getSize())), _timestep(timeStep), _fadeDuration(fadeDuration)
    {
        _overlay.setPosition(0,0);
        _workIt() ;
    }

    void Fader::_workIt()
    {
        sf::Clock total, iteration ;

        while ( _window.isOpen() && total.getElapsedTime() < _fadeDuration )
        {
            sf::Event event ;
            while ( _window.pollEvent(event) )
            {
                switch( event.type )
                {
                case sf::Event::Closed:
                    _window.close() ;
                    break;
                case sf::Event::MouseButtonPressed:
                case sf::Event::MouseButtonReleased:
                case sf::Event::MouseMoved:
                    if ( _eventUpdate )
                        _eventUpdate(event) ;
                    break ;
                }
            }

            if ( iteration.getElapsedTime() >= _timestep )
            {
                const float complete = total.getElapsedTime().asSeconds() / _fadeDuration.asSeconds()  ;
                const unsigned alpha = clamp(_direction == Out ? (complete * 255.0f) : ((1.0f - complete) * 255.0f)) ;
                _overlay.setFillColor(sf::Color(_faded.r, _faded.g, _faded.b, alpha)) ;

                if ( _drawUpdate )
                    _drawUpdate() ;
                else
                    _window.clear() ;

                _window.draw(_overlay) ;
                _window.display() ;
                iteration.restart() ;
            }
        }
    }

# FaderDemo.cpp
This is a bit of a mess, but it should give you an idea of how you can use the class.  Yes, the fade durations are ridiculously long.

    #include <vector>
    #include <cstdlib>
    #include <ctime>
    #include <SFML/Graphics.hpp>
    #include "Fader.h"

    std::vector<sf::RectangleShape> boxes ;
    std::vector<sf::CircleShape> circles ;

    class drawUpdate
    {
    public:
        drawUpdate(sf::RenderWindow& w, std::vector<sf::RectangleShape>& b):_window(w), _boxes(b) {}

        void operator()()
        {
            _window.clear(sf::Color(32, 64, 96)) ;

            sf::RectangleShape box(sf::Vector2f(rand()%25+1, rand()%25+1)) ;
            box.setFillColor( sf::Color(rand()%255, rand()%255, rand()%256, rand()%256)) ;
            box.setPosition(rand()%700+50, rand()%500+50) ;
            boxes.push_back(box) ;

            for ( auto i : boxes )
                _window.draw(i) ;
        }
    private:
        sf::RenderWindow& _window ;
        std::vector<sf::RectangleShape>& _boxes ;
    };

    class Updater
    {
    public:

        Updater(sf::RenderWindow& window, std::vector<sf::RectangleShape> & b, std::vector<sf::CircleShape>& c)
            : _window(window), _boxes(b), _circles(c) {}

        void onEventUpdate(sf::Event & event)
        {
            switch( event.type )
            {
            case sf::Event::MouseMoved:
                {
                    sf::CircleShape circle(static_cast<float>(rand()%50+1)) ;
                    circle.setPosition(event.mouseMove.x, event.mouseMove.y) ;
                    circle.setFillColor(sf::Color(rand()%256, rand()%256, rand()%256, 128)) ;
                    _circles.push_back(circle) ;
                    break ;
                }
            case sf::Event::Closed:
                _window.close() ;
                break ;
            }
        }

        void onDrawUpdate()
        {
            if ( boxes.size() )
                boxes.pop_back() ;

            _window.clear(sf::Color(32, 64, 96)) ;
        
            for ( auto box : boxes )
                _window.draw(box) ;

            for ( auto circle : circles )
                _window.draw(circle) ;
        }

        void ProcessEvents()
        {
            if ( _window.isOpen() )
            {
                sf::Event event ;
                while ( _window.pollEvent(event) )
                    onEventUpdate(event) ;
            }
            onDrawUpdate() ;
            _window.display() ;
        }

    private:
        sf::RenderWindow & _window ;
        std::vector<sf::RectangleShape> & _boxes ;
        std::vector<sf::CircleShape> & _circles ;
    };


    int main()
    {
        srand(time(0)) ;
        sf::RenderWindow window(sf::VideoMode(800, 600), "Fader");

        {
            Fader fader(window, drawUpdate(window, boxes), Fader::Out, nullptr, sf::Color(), sf::seconds(10.0)) ;
        }

        Updater updater(window, boxes, circles) ;
        Fader::onDrawUpdate onDraw(std::bind(&Updater::onDrawUpdate, &updater)) ;
        Fader::onEventUpdate onEvent(std::bind(&Updater::onEventUpdate, &updater, std::placeholders::_1)) ;

        {
            Fader fader(window, onDraw, Fader::In, onEvent, sf::Color(), sf::seconds(10.0)) ;
        }

        while ( window.isOpen() )
            updater.ProcessEvents() ;
    }