# Particle System
Particle systems are great fun to play around with! Using this class one can easily create a particle source that can be moved and "fueled" with more particles. One can also alter the behaviour of all particles by changing the gravity vector, particle speed, particle system shape and particle dissolution rate.

## particle.h
```cpp
#ifndef PARTICLE_H
#define PARTICLE_H

#include <SFML/System/Vector2.hpp>
#include <SFML/System/Clock.hpp>
#include <SFML/Graphics/Color.hpp>
#include <SFML/Graphics/Image.hpp>
#include <SFML/Graphics/Sprite.hpp>
#include <SFML/Graphics/Texture.hpp>
#include <vector>
#include <random>

class Randomizer {
    public:
        Randomizer() : device_(), engine_(device_()){};
        int rnd(int a, int b) {
            std::uniform_int_distribution<int> uni_dist(a, b);
            return uni_dist(engine_);
        };
        double rnd(double a, double b) {
            std::uniform_real_distribution<double> uni_dist(a, b);
            return uni_dist(engine_);
        };
    private:
        std::random_device device_;
        std::default_random_engine engine_;
};

namespace Shape { enum { CIRCLE, SQUARE }; }

struct Particle
{
    sf::Vector2f pos; // Position
    sf::Vector2f vel; // Velocity
    sf::Color color;  // RGBA
};

typedef std::vector<Particle*>::iterator ParticleIterator;

class ParticleSystem
{
public:
    ParticleSystem( int width, int height );
    ~ParticleSystem();

    void fuel( int particles ); // Adds new particles to m_particles
    void update(); // Updates position, velocity and opacity of all particles
    void render(); // Renders all particles onto m_image
    void remove(); // Removes all particles from m_image

    void setPosition( float x, float y ) { m_position.x = x; m_position.y = y; }
    void setGravity( float x, float y ) { m_gravity.x = x; m_gravity.y = y; }
    void setParticleSpeed( float speed ) { m_particleSpeed = speed; }
    void setDissolve( bool enable ) { m_dissolve = enable; }
    void setDissolutionRate( unsigned char rate ) { m_dissolutionRate = rate; }
    void setShape( unsigned char shape ) { m_shape = shape; }

    int getNumberOfParticles() { return m_particles.size(); }
    std::string getNumberOfParticlesString();
    sf::Sprite& getSprite() { return m_sprite; }

private:
    sf::Vector2f    m_position; // Particle origin (pixel co-ordinates)
    sf::Vector2f    m_gravity;  // Affects particle velocities
    sf::Clock   m_clock;    // Used to scale particle motion
    sf::Color   m_transparent;  // sf::Color( 0, 0, 0, 0 )
    sf::Image   m_image;    // See render() and remove()
    sf::Texture m_texture;
    Randomizer  m_randomizer;
    sf::Sprite  m_sprite;   // Connected to m_image
    float       m_particleSpeed;// Pixels per second (at most)
    bool        m_dissolve; // Dissolution enabled?
    unsigned char   m_dissolutionRate;
    unsigned char   m_shape;

    std::vector<Particle*> m_particles;
};

#endif
```

## ParticleSystem.cpp
```cpp
#include "ParticleSystem.h"
#include <sstream>

ParticleSystem::ParticleSystem( int width, int height )
{
    m_transparent = sf::Color( 0, 0, 0, 0 );
    m_image.create( width, height, m_transparent );
    m_texture.loadFromImage(m_image);
    m_sprite = sf::Sprite(m_texture);
    m_position.x    = 0.5f * width;
    m_position.y    = 0.5f * height;
    m_particleSpeed = 20.0f;
    m_dissolve  = false;
    m_dissolutionRate = 4;
    m_shape     = Shape::CIRCLE;
}

ParticleSystem::~ParticleSystem()
{
    for( ParticleIterator it = m_particles.begin(); it != m_particles.end(); it++ )
    {
        delete *it;
    }
}

void ParticleSystem::fuel( int particles )
{
    float angle;
    Particle* particle;
    for( int i = 0; i < particles; i++ )
    {
        particle = new Particle();
        particle->pos.x = m_position.x;
        particle->pos.y = m_position.y;

        switch( m_shape )
        {
        case Shape::CIRCLE:
            angle = m_randomizer.rnd(0.0f, 6.2832f);
            particle->vel.x = m_randomizer.rnd(0.0f, cos( angle ));
            particle->vel.y = m_randomizer.rnd(0.0f, sin( angle ));
            break;
        case Shape::SQUARE:
            particle->vel.x = m_randomizer.rnd(-1.0f, 1.0f);
            particle->vel.y = m_randomizer.rnd(-1.0f, 1.0f);
            break;
        default:
            particle->vel.x = 0.5f; // Easily detected
            particle->vel.y = 0.5f; // Easily detected
        }

        if( particle->vel.x == 0.0f && particle->vel.y == 0.0f )
        {
            delete particle;
            continue;
        }
        particle->color.r = m_randomizer.rnd(0, 255);
        particle->color.g = m_randomizer.rnd(0, 255);
        particle->color.b = m_randomizer.rnd(0, 255);
        particle->color.a = 255;
        m_particles.push_back( particle );
    }
}

void ParticleSystem::update()
{
    float time = m_clock.restart().asSeconds();

    for( ParticleIterator it = m_particles.begin(); it != m_particles.end(); it++ )
    {
        (*it)->vel.x += m_gravity.x * time;
        (*it)->vel.y += m_gravity.y * time;

        (*it)->pos.x += (*it)->vel.x * time * m_particleSpeed;
        (*it)->pos.y += (*it)->vel.y * time * m_particleSpeed;

        if( m_dissolve ) (*it)->color.a -= m_dissolutionRate;

        if( (*it)->pos.x > m_image.getSize().x || (*it)->pos.x < 0 || (*it)->pos.y > m_image.getSize().y || (*it)->pos.y < 0 || (*it)->color.a < 10 )
        {
            delete (*it);
            it = m_particles.erase( it );
            if( it == m_particles.end() ) return;
        }
    }
}

void ParticleSystem::render()
{
    for( ParticleIterator it = m_particles.begin(); it != m_particles.end(); it++ )
    {
        m_image.setPixel( (int)(*it)->pos.x, (int)(*it)->pos.y, (*it)->color );
    }
    m_texture.update(m_image);
}

void ParticleSystem::remove()
{
    for( ParticleIterator it = m_particles.begin(); it != m_particles.end(); it++ )
    {
        m_image.setPixel( (int)(*it)->pos.x, (int)(*it)->pos.y, m_transparent );
    }
}

std::string ParticleSystem::getNumberOfParticlesString()
{
    std::ostringstream oss;
    oss << m_particles.size();
    return oss.str();
}
```

## main.cpp 
```cpp
#include <SFML/Window.hpp>
#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/Graphics/Text.hpp>
#include "ParticleSystem.h"

int main()
{
    int width   = 640;
    int height  = 480;

    sf::RenderWindow window( sf::VideoMode( width, height, 32 ), "Inside the Particle Storm" );
    window.setVerticalSyncEnabled( true );
    //const sf::Input& input = window.getInput();
    sf::Event events;
    sf::Font font;
    if( !font.loadFromFile( "/usr/share/fonts/truetype/yudit.ttf" ) ) return 1;
    sf::Text text( "", font, 10 );
    text.setPosition( 10.0f, 10.0f );

    ParticleSystem particleSystem( width, height );
    //particleSystem.setPosition( width/2, height/2 );
    //particleSystem.setGravity( 1.0f, 1.0f );
    //particleSystem.setParticleSpeed( 80.0f );
    particleSystem.setDissolve( true );
    particleSystem.setDissolutionRate( 1 );
    particleSystem.setShape( Shape::CIRCLE );

    particleSystem.fuel( 1000 );

    float xpos = 320.0f;
    float ypos = 240.0f;
    float xgrv = 0.0f;
    float ygrv = 0.0f;

    while( window.isOpen() )
    {
        while (window.pollEvent(events))
            switch (events.type) {
                case sf::Event::Closed:
                    window.close(); break;
                case sf::Event::KeyPressed:
                    if(events.key.code == sf::Keyboard::Escape )
                        window.close();
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::Space ) )
                        particleSystem.fuel( 200/* * window.getFrameTime() */);
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::A ) )
                        particleSystem.setPosition( --xpos, ypos );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::D ) )
                        particleSystem.setPosition( ++xpos, ypos );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::W ) )
                        particleSystem.setPosition( xpos, --ypos );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::S ) )
                        particleSystem.setPosition( xpos, ++ypos );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::Left ) )
                        particleSystem.setGravity( --xgrv * 0.1f, ygrv * 0.1f);
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::Right ) )
                        particleSystem.setGravity( ++xgrv * 0.1f, ygrv * 0.1f );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::Up ) )
                        particleSystem.setGravity( xgrv * 0.1f, --ygrv * 0.1f );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::Down ) )
                        particleSystem.setGravity( xgrv * 0.1f, ++ygrv * 0.1f );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::G ) )
                        particleSystem.setGravity( 0.0f, 0.0f );
                    if( sf::Keyboard::isKeyPressed( sf::Keyboard::P ) )
                        particleSystem.setPosition( 320.0f, 240.0f );
                    break;
            }

        particleSystem.remove();
        particleSystem.update();
        particleSystem.render();

        text.setString( particleSystem.getNumberOfParticlesString() );
        window.clear( sf::Color::Black );
        window.draw( particleSystem.getSprite() );
        window.draw( text );
        window.display();
    }

    return 0;
}
```

## Credits
The original article was written by the user Lillebror.