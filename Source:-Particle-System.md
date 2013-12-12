# Particle System
Particle systems are great fun to play around with! Using this class one can easily create a particle source that can be moved and "fueled" with more particles. One can also alter the behaviour of all particles by changing the gravity vector, particle speed, particle system shape and particle dissolution rate.

## particle.h
```cpp
#ifndef PARTICLE_H
#define PARTICLE_H

#include <SFML\System\Vector2.hpp>
#include <SFML\System\Clock.hpp>
#include <SFML\System\Randomizer.hpp>
#include <SFML\Graphics\Color.hpp>
#include <SFML\Graphics\Image.hpp>
#include <SFML\Graphics\Sprite.hpp>
#include <vector>

namespace Shape { enum { CIRCLE, SQUARE }; }

struct Particle
{
	sf::Vector2f pos; // Position
	sf::Vector2f vel; // Velocity
	sf::Color color;  // RGBA
};

typedef std::list<Particle*>::iterator ParticleIterator;

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
	sf::Vector2f	m_position;	// Particle origin (pixel co-ordinates)
	sf::Vector2f	m_gravity;	// Affects particle velocities
	sf::Randomizer	m_randomizer;	// Randomizes particle velocities
	sf::Clock	m_clock;	// Used to scale particle motion
	sf::Color	m_transparent;	// sf::Color( 0, 0, 0, 0 )
	sf::Image	m_image;	// See render() and remove()
	sf::Sprite	m_sprite;	// Connected to m_image
	float		m_particleSpeed;// Pixels per second (at most)
	bool		m_dissolve;	// Dissolution enabled?
	unsigned char	m_dissolutionRate;
	unsigned char	m_shape;

	std::vector<Particle*> m_particles;
};

#endif
```

## particle.cpp
```cpp
#include "particle.h"
#include <sstream>

ParticleSystem::ParticleSystem( int width, int height )
{
	m_transparent = sf::Color( 0, 0, 0, 0 );
	m_image.Create( width, height, m_transparent );
	m_sprite.SetImage( m_image );

	m_position.x	= 0.5f * width;
	m_position.y	= 0.5f * height;
	m_particleSpeed	= 20.0f;
	m_dissolve	= false;
	m_dissolutionRate = 4;
	m_shape		= Shape::CIRCLE;
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
			angle = m_randomizer.Random( 0.0f, 6.2832f );
			particle->vel.x = m_randomizer.Random( 0.0f, cos( angle ) );
			particle->vel.y = m_randomizer.Random( 0.0f, sin( angle ) );
			break;
		case Shape::SQUARE:
			particle->vel.x = m_randomizer.Random( -1.0f, 1.0f );
			particle->vel.y = m_randomizer.Random( -1.0f, 1.0f );
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

		particle->color.r = m_randomizer.Random( 0, 255 );
		particle->color.g = m_randomizer.Random( 0, 255 );
		particle->color.b = m_randomizer.Random( 0, 255 );
		particle->color.a = 255;
		m_particles.push_back( particle );
	}
}

void ParticleSystem::update()
{
	float time = m_clock.GetElapsedTime();
	m_clock.Reset();

	for( ParticleIterator it = m_particles.begin(); it != m_particles.end(); it++ )
	{
		(*it)->vel.x += m_gravity.x * time;
		(*it)->vel.y += m_gravity.y * time;

		(*it)->pos.x += (*it)->vel.x * time * m_particleSpeed;
		(*it)->pos.y += (*it)->vel.y * time * m_particleSpeed;

		if( m_dissolve ) (*it)->color.a -= m_dissolutionRate;

		if( (*it)->pos.x > m_image.GetWidth() || (*it)->pos.x < 0 || (*it)->pos.y > m_image.GetHeight() || (*it)->pos.y < 0 || (*it)->color.a < 10 )
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
		m_image.SetPixel( (int)(*it)->pos.x, (int)(*it)->pos.y, (*it)->color );
	}
}

void ParticleSystem::remove()
{
	for( ParticleIterator it = m_particles.begin(); it != m_particles.end(); it++ )
	{
		m_image.SetPixel( (int)(*it)->pos.x, (int)(*it)->pos.y, m_transparent );
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
#include "particle.h"

int main()
{
	int width	= 640;
	int height	= 480;

	sf::RenderWindow window( sf::VideoMode( width, height, 32 ), "Inside the Particle Storm" );
	window.UseVerticalSync( true );
	const sf::Input& input = window.GetInput();
	sf::Event events;
	sf::Font font;
	if( !font.LoadFromFile( "arial.ttf" ) ) return 1;
	sf::Text text( "", font, 10 );
	text.SetPosition( 10.0f, 10.0f );

	ParticleSystem particleSystem( width, height );
	//particleSystem.setPosition( width/2, height/2 );
	//particleSystem.setGravity( 1.0f, 1.0f );
	//particleSystem.setParticleSpeed( 80.0f );
	//particleSystem.setDissolve( true );
	//particleSystem.setDissolutionRate( 4 );
	//particleSystem.setShape( Shape::SQUARE );

	particleSystem.fuel( 1000 );

	float xpos = 320.0f;
	float ypos = 240.0f;
	float xgrv = 0.0f;
	float ygrv = 0.0f;

	while( window.IsOpened() )
	{
		window.GetEvent( events );
		if( events.Type == sf::Event::Closed )
			window.Close();
		if( events.Type == sf::Event::KeyPressed && events.Key.Code == sf::Key::Escape )
			window.Close();

		if( input.IsKeyDown( sf::Key::Space ) )
			particleSystem.fuel( 200 * window.GetFrameTime() );
		if( input.IsKeyDown( sf::Key::A ) )
			particleSystem.setPosition( --xpos, ypos );
		if( input.IsKeyDown( sf::Key::D ) )
			particleSystem.setPosition( ++xpos, ypos );
		if( input.IsKeyDown( sf::Key::W ) )
			particleSystem.setPosition( xpos, --ypos );
		if( input.IsKeyDown( sf::Key::S ) )
			particleSystem.setPosition( xpos, ++ypos );
		if( input.IsKeyDown( sf::Key::Left ) )
			particleSystem.setGravity( --xgrv * 0.1f, ygrv * 0.1f);
		if( input.IsKeyDown( sf::Key::Right ) )
			particleSystem.setGravity( ++xgrv * 0.1f, ygrv * 0.1f );
		if( input.IsKeyDown( sf::Key::Up ) )
			particleSystem.setGravity( xgrv * 0.1f, --ygrv * 0.1f );
		if( input.IsKeyDown( sf::Key::Down ) )
			particleSystem.setGravity( xgrv * 0.1f, ++ygrv * 0.1f );
		if( input.IsKeyDown( sf::Key::G ) )
			particleSystem.setGravity( 0.0f, 0.0f );
		if( input.IsKeyDown( sf::Key::P ) )
			particleSystem.setPosition( 320.0f, 240.0f );

		particleSystem.remove();
		particleSystem.update();
		particleSystem.render();
		
		text.SetString( particleSystem.getNumberOfParticlesString() );
		window.Clear( sf::Color::Black );
		window.Draw( particleSystem.getSprite() );
		window.Draw( text );
		window.Display();
	}

	return 0;
}
```

## Credits
The original article was written by the user Lillebror.