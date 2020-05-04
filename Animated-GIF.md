A small class to display animated GIFs. This is a prove of concept; you'll probably don't just want to copy and paste the code. It makes use of [stb_image.h](https://github.com/nothings/stb), which SFML also uses to load images.

### AnimatedGIF.h
```cpp
#ifndef __ANIMATEDGIF_H__
#define __ANIMATEDGIF_H__

#include <SFML/Graphics.hpp>

#include <vector>
#include <tuple>

class AnimatedGIF
{
    public:
        AnimatedGIF(const char* filename);

        const sf::Vector2i& getSize(void);
        void update(sf::Sprite& sprite);

    private:
        sf::Vector2i size;
        sf::Clock clock;
        sf::Time startTime;
        sf::Time totalDelay;
        std::vector<std::tuple<int, sf::Texture>> frames;
        std::vector<std::tuple<int, sf::Texture>>::iterator frameIter;
};

#endif
```
### AnimatedGIF.cpp
```cpp
#include "AnimatedGIF.h"

#define STB_IMAGE_IMPLEMENTATION
#include <stb/stb_image.h>
#define STBI_GIF_ONLY

AnimatedGIF::AnimatedGIF(const char* filename)
{
    FILE *f = stbi__fopen(filename, "rb");
    stbi__context s;
    stbi__start_file(&s, f);

    int *delays;
    int z = 0, comp = 0;

    void *pixels = stbi__load_gif_main(&s, &delays, &size.x, &size.y, &z, &comp, STBI_rgb_alpha);

    sf::Image image;
    int step = size.x * size.y * 4;

    for (int i = 0; i < z; i++)
    {
        image.create(size.x, size.y, (const sf::Uint8*) pixels + step * i);

        sf::Texture texture;
        texture.loadFromImage(image);

        frames.push_back(std::tuple<int, sf::Texture>(delays[i], texture));
    }

    frameIter = frames.begin();
    
    stbi_image_free(pixels);

    totalDelay = sf::Time::Zero;
    startTime = clock.getElapsedTime();
}

const sf::Vector2i&
AnimatedGIF::getSize()
{
    return size;
}

void
AnimatedGIF::update(sf::Sprite& sprite)
{
    sf::Time delay = sf::milliseconds(std::get<0>(*frameIter));

    while (startTime + totalDelay + delay < clock.getElapsedTime())
    {
        frameIter++;
        if (frameIter == frames.end()) frameIter = frames.begin();
        totalDelay += delay;
        delay = sf::milliseconds(std::get<0>(*frameIter));
    }

    sf::Texture &texture = std::get<1>(*frameIter);
    sprite.setTexture(texture);
}
```
### main.cpp
```cpp
#include <iostream>
#include <SFML/Graphics.hpp>

#include "AnimatedGIF.h"

int main(int argc, char** argv) 
{
    if (argc < 2)
    {
        std::cerr << "No argument" << std::endl;
        return 1;
    }

    AnimatedGIF gif(argv[1]);
    sf::Vector2i size = gif.getSize();

    sf::RenderWindow window(sf::VideoMode(size.x, size.y), "gifdemo");
    window.setFramerateLimit(60);

    sf::Sprite sprite;

    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        gif.update(sprite);
        window.draw(sprite);
        window.display();
    }

    return EXIT_SUCCESS;
}
```