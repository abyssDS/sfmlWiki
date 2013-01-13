# AnimatedSprite class

## Summary 
This is a class for SFML 2 that provides a easy interface to animate Sprites. The design closely follows the design of the sf::Sprite class.

Class written by Foaly. You can use it in any way you want. Just a little note that the class was written by me would be nice :)

##Usage
The usage is quiet simple. First you create a Animation. The Animation class is essentially a std::vector of sf::IntRect and a sf::Texture reference. So you have to provide a Spritesheet and push back your intrects. Then you create a AnimatedSprite object and provide it with a Animation. The AnimatedSprite class inherits from both sf::Drawable and sf::Transfomable, so it behaves much like a regular sprite. The only thing you have to do is to call the update(sf::Time deltaTime) member on every iteration with the time passed since the last iteration (this also allows you to do effects like slow motion without changing the frame time). Of course the AnimatedSprite class also provides members to pause, stop and play the Animation.
Note that both classes work with references, so you have to keep the sf::Texture and the Animation alive as long as you need AnimatedSprite.
If all of this to confusing to you then take a look at the example and play with it.

## Example
Here is a short example of how to use the classes. 

Spritesheet: ![Spritesheet](http://i.imgur.com/355uI.png)
```cpp
#include <SFML/Graphics.hpp>
#include "AnimatedSprite.hpp"

int main()
{
    sf::RenderWindow window(sf::VideoMode(200, 200), "Animations!");

    // load texture
    sf::Texture texture;
    texture.loadFromFile("Gnome.png");

    // push frames
    Animation walkingAnimation;
    walkingAnimation.setSpriteSheet(texture);
    walkingAnimation.pushBackFrame(sf::IntRect(0, 0, 53, 95));
    walkingAnimation.pushBackFrame(sf::IntRect(53, 0, 57, 97));
    walkingAnimation.pushBackFrame(sf::IntRect(0, 0, 53, 95));
    walkingAnimation.pushBackFrame(sf::IntRect(110, 0, 57, 97));

    // set up AnimatesSprite
    AnimatedSprite animatedSprite(sf::seconds(0.2));
    animatedSprite.setAnimation(walkingAnimation);
//    animatedSprite.setColor(sf::Color::Red);
    animatedSprite.setPosition(100, 100);

    sf::Clock frameClock;

    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                window.close();
            if (event.type == sf::Event::KeyPressed && event.key.code == sf::Keyboard::Escape)
                window.close();
            if (event.type == sf::Event::KeyPressed && event.key.code == sf::Keyboard::P)
            {
                if(animatedSprite.isPlaying())
                    animatedSprite.pause();
                else
                    animatedSprite.play();
            }
        }

        // update AnimatedSprite
        animatedSprite.update(frameClock.restart());

        window.clear();
        window.draw(animatedSprite);
        window.display();
    }

    return 0;
}
```

##Source
### Animation.hpp
```cpp
#ifndef ANIMATION_INCLUDE
#define ANIMATION_INCLUDE

// Class written by Foaly

#include <vector>
#include <SFML/Graphics/Rect.hpp>
#include <SFML/Graphics/Texture.hpp>

class Animation
{
public:
    Animation();

    void pushBackFrame(sf::IntRect rect);
    void setSpriteSheet(sf::Texture& texture);
    const sf::Texture* getSpriteSheet() const;
    std::size_t getSize();
    sf::IntRect& getFrame(const std::size_t n);

private:
    std::vector<sf::IntRect> m_frames;
    const sf::Texture* m_texture;
};

#endif // ANIMATION_INCLUDE
```
### Animation.cpp
```cpp
#include "Animation.hpp"

Animation::Animation() : m_texture(NULL)
{

}

void Animation::pushBackFrame(sf::IntRect rect)
{
    m_frames.push_back(rect);
}

void Animation::setSpriteSheet(sf::Texture& texture)
{
    m_texture = &texture;
}

const sf::Texture* Animation::getSpriteSheet() const
{
    return m_texture;
}

std::size_t Animation::getSize()
{
    return m_frames.size();
}

sf::IntRect& Animation::getFrame(const std::size_t n)
{
    return m_frames[n];
}

```

### AnimatedSprite.hpp
```cpp
#ifndef ANIMATEDSPRITE_INCLUDE
#define ANIMATEDSPRITE_INCLUDE

// Class written by Foaly

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/System/Time.hpp>
#include <SFML/Graphics/Drawable.hpp>
#include <SFML/Graphics/Transformable.hpp>
#include <SFML/System/Vector2.hpp>

#include "Animation.hpp"

class AnimatedSprite : public sf::Drawable, public sf::Transformable
{
public:
    AnimatedSprite(sf::Time frameTime = sf::seconds(0.2), bool paused = false, bool looped = true);

    void update(sf::Time deltaTime);
    void setAnimation(Animation& animation);
    void setFrameTime(sf::Time time);
    void play();
    void pause();
    void stop();
    void setLooped(bool looped);
    void setColor(const sf::Color& color);
    sf::FloatRect getLocalBounds() const;
    sf::FloatRect getGlobalBounds() const;
    bool isLooped() const;
    bool isPlaying() const;
    sf::Time getFrameTime() const;



private:
    Animation* m_animation;
    sf::Time m_frameTime;
    sf::Time m_currentTime;
    std::size_t m_currentFrame;
    bool m_isPaused;
    bool m_isLooped;
    const sf::Texture* m_texture;
    sf::Vertex m_vertices[4];

    virtual void draw(sf::RenderTarget& target, sf::RenderStates states) const;

};

#endif // ANIMATEDSPRITE_INCLUDE
```

### AnimatedSprite.cpp
```cpp
#include "AnimatedSprite.hpp"

AnimatedSprite::AnimatedSprite(sf::Time frameTime, bool paused, bool looped) :
    m_animation(NULL), m_currentFrame(0), m_isPaused(paused), m_isLooped(looped), m_texture(NULL)
{
    m_frameTime = frameTime;
}

void AnimatedSprite::setAnimation(Animation& animation)
{
    m_animation = &animation;
    m_texture = m_animation->getSpriteSheet();
}

void AnimatedSprite::play()
{
    if(m_isPaused)
    {
        m_currentTime = sf::Time::Zero;
        m_isPaused = false;
    }
}

void AnimatedSprite::pause()
{
    m_isPaused = true;
}

void AnimatedSprite::stop()
{
    m_isPaused = true;
    m_currentFrame = 0;
}

void AnimatedSprite::setLooped(bool looped)
{
    m_isLooped = looped;
}

void AnimatedSprite::setColor(const sf::Color& color)
{
    // Update the vertices' color
    m_vertices[0].color = color;
    m_vertices[1].color = color;
    m_vertices[2].color = color;
    m_vertices[3].color = color;
}
sf::FloatRect AnimatedSprite::getLocalBounds() const
{
    sf::IntRect rect = m_animation->getFrame(m_currentFrame);

    float width = static_cast<float>(std::abs(rect.width));
    float height = static_cast<float>(std::abs(rect.height));

    return sf::FloatRect(0.f, 0.f, width, height);
}

sf::FloatRect AnimatedSprite::getGlobalBounds() const
{
    return getTransform().transformRect(getLocalBounds());
}

bool AnimatedSprite::isLooped() const
{
    return m_isLooped;
}

bool AnimatedSprite::isPlaying() const
{
    return !m_isPaused;
}

sf::Time AnimatedSprite::getFrameTime() const
{
    return m_frameTime;
}


void AnimatedSprite::update(sf::Time deltaTime)
{
    // if not paused and we have a valid animation
    if(!m_isPaused && m_animation)
    {
        // add delta time
        m_currentTime += deltaTime;

        // if current time is bigger then the frame time advance one frame
        if(m_currentTime >= m_frameTime)
        {
            // reset time
            m_currentTime = sf::Time::Zero;

            // get next Frame index
            if(m_currentFrame + 1 < m_animation->getSize())
                m_currentFrame++;
            else
            {
                // animation has ended
                m_currentFrame = 0; // reset to start

                if(!m_isLooped)
                {
                    m_isPaused = true;
                }

            }

            //calculate new vertex positions and texture coordiantes
            sf::IntRect rect = m_animation->getFrame(m_currentFrame);

            m_vertices[0].position = sf::Vector2f(0, 0);
            m_vertices[1].position = sf::Vector2f(0, rect.height);
            m_vertices[2].position = sf::Vector2f(rect.width, rect.height);
            m_vertices[3].position = sf::Vector2f(rect.width, 0);

            float left = static_cast<float>(rect.left);
            float right = left + rect.width;
            float top = static_cast<float>(rect.top);
            float bottom = top + rect.height;

            m_vertices[0].texCoords = sf::Vector2f(left, top);
            m_vertices[1].texCoords = sf::Vector2f(left, bottom);
            m_vertices[2].texCoords = sf::Vector2f(right, bottom);
            m_vertices[3].texCoords = sf::Vector2f(right, top);
        }
    }
}

void AnimatedSprite::draw(sf::RenderTarget& target, sf::RenderStates states) const
{
    if (m_animation && m_texture)
    {
        states.transform *= getTransform();
        states.texture = m_texture;
        target.draw(m_vertices, 4, sf::Quads, states);
    }
}
```
