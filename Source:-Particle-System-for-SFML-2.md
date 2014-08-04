# Particle System for SFML 2
Particle systems are great fun to play around with! Using this class one can easily create a particle source that can be moved and "fueled" with more particles. One can also alter the behaviour of all particles by changing the gravity vector, particle speed, particle system shape and particle dissolution rate.

This code has been updated from the previous version to run with SFML 2.1 as well as a few tweeks.
The original code was taken from [[This Article|Source:-Particle-System]]

## particle.h
```cpp
#pragma once

#ifndef PARTICLE_H
#define PARTICLE_H

#include <list>
#include <memory>
#include <random>
#include <sstream>
#include <vector>
#include <SFML/Graphics.hpp>

/* Enum for particle distribution type */
namespace Shape { enum { CIRCLE, SQUARE }; }

/* Particle Structure */
struct Particle : public sf::Drawable
{
  /* Data Members */

  sf::Vertex drawVertex; /*< To replace pos */
  sf::Vector2f vel; // Velocity

  /* Member Functions */

  virtual void draw(sf::RenderTarget &target, sf::RenderStates states) const
    { target.draw(&drawVertex, 1, sf::Points, states); }

};

/* Typedefs */
typedef std::uniform_real_distribution<> UniRealDist;
typedef std::uniform_int_distribution<> UniIntDist;
typedef std::shared_ptr<Particle> ParticlePtr;

class ParticleSystem : public sf::Drawable
{
public:

  /* Constructors/Destructor */

  ParticleSystem(sf::Vector2u canvasSize);
  ~ParticleSystem(void);

  /* Getters and Setters */

  const int getDissolutionRate(void) const { return m_dissolutionRate; }
  const int getNumberOfParticles(void) const { return m_particles.size(); }
  const float getParticleSpeed(void) const { return m_particleSpeed; }
  const std::string getNumberOfParticlesString(void) const;

  void setCanvasSize(sf::Vector2u newSize) { m_canvasSize = newSize; }
  void setDissolutionRate(sf::Uint8 rate) { m_dissolutionRate = rate; }
  void setDissolve(void) { m_dissolve = !m_dissolve; }
  void setDistribution(void) { m_shape = (m_shape + 1) % 2; }
  void setGravity(float x, float y) { m_gravity.x = x; m_gravity.y = y; }
  void setGravity(sf::Vector2f gravity) { m_gravity = gravity; }
  void setParticleSpeed(float speed) { m_particleSpeed = speed; }
  void setPosition(float x, float y) { m_startPos.x = x; m_startPos.y = y; }
  void setPosition(sf::Vector2f position) { m_startPos = position; }
  void setShape(sf::Uint8 shape) { m_shape = shape; }

  /* Member Functions */

  virtual void draw(sf::RenderTarget &target, sf::RenderStates states) const;

  void fuel(int particles);     /*< Adds new particles */
  void update(float deltaTime); /*< Updates particles */

private:

  /* Data Members */

  bool        m_dissolve;       /*< Dissolution enabled? */
  float       m_particleSpeed;  /*< Pixels per second (at most) */

  sf::Color   m_transparent;    /*< sf::Color(0, 0, 0, 0) */

  sf::Uint8   m_dissolutionRate;/*< Rate particles disolve */
  sf::Uint8   m_shape;          /*< Shape of distribution */

  sf::Vector2f    m_gravity;    /*< Influences particle velocities */
  sf::Vector2f    m_startPos;   /*< Particle origin */
  sf::Vector2u    m_canvasSize;/*< Limits of particle travel */

  /* Container */
  std::vector<ParticlePtr> m_particles;

};

#endif
```

## particle.cpp
```cpp
#include "particle.h"

/************************************************************/
ParticleSystem::ParticleSystem(sf::Vector2u canvasSize) :
  m_dissolve(false),
  m_particleSpeed(100.0f),
  m_transparent(sf::Color(0,0,0,0)),
  m_dissolutionRate(4),
  m_shape(Shape::CIRCLE),
  m_gravity(sf::Vector2f(0.0f,0.0f)),
  m_canvasSize(canvasSize)
{
  m_startPos =
    sf::Vector2f(
      static_cast<float>(canvasSize.x) / 2,
      static_cast<float>(canvasSize.y) / 2);
}

/************************************************************/
ParticleSystem::~ParticleSystem(void)
{
  /* Clear vector */
  m_particles.clear();
}

/************************************************************/
void ParticleSystem::draw(
  sf::RenderTarget& target, sf::RenderStates states) const
{
  for(const auto& item : m_particles)
    target.draw(&item.get()->drawVertex, 1, sf::Points);
  return;
}

/************************************************************/
void ParticleSystem::fuel(int particles)
{
  for(int i = 0; i < particles; i++)
  {
    /* Generate a new particle and put it at the generation point */
    Particle* particle;
    particle = new Particle();
    particle->drawVertex.position.x = m_startPos.x;
    particle->drawVertex.position.y = m_startPos.y;

    /* Randomizer initialization */
    std::random_device rd;
    std::mt19937 gen(rd());

    switch(m_shape)
    {
      case Shape::CIRCLE:
      {
        /* Generate a random angle */
        UniRealDist randomAngle(0.0f, (2.0f * 3.14159265));
        float angle = randomAngle(gen);

        /* Use the random angle as a thrust vector for the particle */
        UniRealDist randomAngleCos(0.0f, cos(angle));
        UniRealDist randomAngleSin(0.0f, sin(angle));
        particle->vel.x = randomAngleCos(gen);
        particle->vel.y = randomAngleSin(gen);

        break;
      }
      case Shape::SQUARE:
      {
        /* Square generation */
        UniRealDist randomSide(-1.0f, 1.0f);
        particle->vel.x = randomSide(gen);
        particle->vel.y = randomSide(gen);

        break;
      }
      default:
      {
        particle->vel.x = 0.5f; // Easily detected
        particle->vel.y = 0.5f; // Easily detected
      }
    }

    /* We don't want lame particles. Reject, start over. */
    if(particle->vel.x == 0.0f && particle->vel.y == 0.0f )
    {
      delete particle;
      continue;
    }

    /* Randomly change the colors of the particles */
    UniIntDist randomColor(0, 255);
    particle->drawVertex.color.r = randomColor(gen);
    particle->drawVertex.color.g = randomColor(gen);
    particle->drawVertex.color.b = randomColor(gen);
    particle->drawVertex.color.a = 255;

    m_particles.push_back(ParticlePtr(particle));
  }

  return;
}

/************************************************************/
const std::string ParticleSystem::getNumberOfParticlesString(void) const
{
    std::ostringstream oss;
    oss << m_particles.size();
    return oss.str();
}

/************************************************************/
void ParticleSystem::update(float deltaTime)
{
  /* Run through each particle and apply our system to it */
  for(auto it = m_particles.begin(); it != m_particles.end(); it++)
  {
    /* Apply Gravity */
    (*it)->vel.x += m_gravity.x * deltaTime;
    (*it)->vel.y += m_gravity.y * deltaTime;

    /* Apply thrust */
    (*it)->drawVertex.position.x += (*it)->vel.x * deltaTime * m_particleSpeed;
    (*it)->drawVertex.position.y += (*it)->vel.y * deltaTime * m_particleSpeed;

    /* If they are set to disolve, disolve */
    if(m_dissolve) (*it)->drawVertex.color.a -= m_dissolutionRate;

    if((*it)->drawVertex.position.x > m_canvasSize.x
       || (*it)->drawVertex.position.x < 0
       || (*it)->drawVertex.position.y > m_canvasSize.y
       || (*it)->drawVertex.position.y < 0
       || (*it)->drawVertex.color.a < 10)
    {
      it = m_particles.erase(it);
      if(it == m_particles.end()) return;
    }
  }

  return;
}
```

## main.cpp 
```cpp
#include <sstream>
#include <SFML/Graphics.hpp>
#include "particle.h"

int main()
{
  /* Define desired resolution and open a window */
  sf::VideoMode videoMode(800, 800);
  sf::RenderWindow window(
    sf::VideoMode(videoMode),
    "Inside the Particle Storm");
  window.setVerticalSyncEnabled(true);

  /* Load a font and setup some texty-type stuff */
  sf::Font font;
  if(!font.loadFromFile("Fixedsys500c.ttf"))
    return 1;
  sf::Text text("", font, 12);
  text.setPosition(
    static_cast<float>(window.getSize().x) * 0.01f,
    static_cast<float>(window.getSize().y) * 0.01f);

  /* Create the particle system and give it some fuel */
  ParticleSystem particleSystem(window.getSize());
  particleSystem.fuel(1000);

  /* Let's make a clock and junk for timing stuff! */
  sf::Clock timer;
  const sf::Uint32 UPDATE_STEP = 20;
  const sf::Uint32 MAX_UPDATE_SKIP = 5;
  sf::Uint32 nextUpdate = timer.getElapsedTime().asMilliseconds();

  /* For some fancy mouse stuff */
  sf::Vector2f lastMousePos(static_cast<sf::Vector2f>(window.getSize()));

  /* Main Loop */
  while(window.isOpen())
  {
    /* Init a skipped frame counter */
    sf::Uint32 frameSkips = 0;

    /* Throttle handlers and update to every UPDATE_STEP */
    while(timer.getElapsedTime().asMilliseconds() > nextUpdate
          && frameSkips < MAX_UPDATE_SKIP)
    {
      /* Handle events */
      sf::Event event;
      while(window.pollEvent(event))
      {
        switch(event.type)
        {
          case(sf::Event::Closed):
          {
            window.close();
            break;
          }
          case(sf::Event::KeyPressed):
          {
            if(event.key.code == sf::Keyboard::Escape)
              window.close();
            if(event.key.code == sf::Keyboard::Space)
              particleSystem.setDissolve();
            if(event.key.code == sf::Keyboard::A)
            {
              if(particleSystem.getDissolutionRate() > 0)
                particleSystem.setDissolutionRate(
                  particleSystem.getDissolutionRate() - 1);
            }
            if(event.key.code == sf::Keyboard::S)
              particleSystem.setDissolutionRate(
                particleSystem.getDissolutionRate() + 1);
            if(event.key.code == sf::Keyboard::W)
              particleSystem.setParticleSpeed(
                particleSystem.getParticleSpeed()
                + particleSystem.getParticleSpeed() * 0.1);
            if(event.key.code == sf::Keyboard::Q
              && particleSystem.getParticleSpeed() > 0)
              particleSystem.setParticleSpeed(
                particleSystem.getParticleSpeed()
                - particleSystem.getParticleSpeed() * 0.1);
            if(event.key.code == sf::Keyboard::E)
              particleSystem.setDistribution();
            break;
          }
          default:
            break;
        }
      }

      /* Mouse Input */
      /* Set the position to match the mouse location */
      sf::Vector2f mousePos =
        static_cast<sf::Vector2f>(sf::Mouse::getPosition(window));

      /* Update Particle Emitter to Mouse Position */
      if(mousePos.x > 0
        || mousePos.y > 0
        || mousePos.x < window.getSize().x
        || mousePos.y < window.getSize().y)
        particleSystem.setPosition(mousePos);

      /* Mouse Clicks */
      if(sf::Mouse::isButtonPressed(sf::Mouse::Left))
        particleSystem.fuel(25);
      if(sf::Mouse::isButtonPressed(sf::Mouse::Right))
      {
        sf::Vector2f newGravity = lastMousePos - mousePos;
        newGravity *= 0.75f;
        particleSystem.setGravity(newGravity);
      }
      if(sf::Mouse::isButtonPressed(sf::Mouse::Middle))
        particleSystem.setGravity(0.0f, 0.0f);

      /* Update Last Mouse Position */
      lastMousePos = mousePos;

      /* Push Diag Text */
      std::ostringstream buffer;
      buffer << "Q/W to Decrease/Increase Particle Speed\n"
             << "A/S to Decrease/Increase Decay Rate\n"
             << "Right Click+Drag to Shift Gravity\n"
             << "E to Change Distribution Type\n"
             << "Middle Click clears Gravity\n"
             << "Left Click to Add\n"
             << "Particles: " << particleSystem.getNumberOfParticles();
      text.setString(buffer.str());

      /* Update particle system */
      particleSystem.update(static_cast<float>(UPDATE_STEP) / 1000);

      frameSkips++;
      nextUpdate += UPDATE_STEP;
    }

    /* Draw particle system and text */
    window.clear(sf::Color::Black);
    window.draw(text);
    window.draw(particleSystem);
    window.display();
  }

  return 0;
}
```

## Credits
The original article was written by the user Lillebror.

Updated by the user bloodythorn.