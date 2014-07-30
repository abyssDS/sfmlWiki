This is a simple example implementation of a shader that produces a heat haze effect in SFML:

![Example Output](http://i.imgur.com/FOScjL8.gif)

If you don't want to generate the distortion map image using the code, you will have to do so with an external program.
* It has to contain coherent noise values in each channel
* It should contain values ranging from minimum to the maximum intensity
* It should be seamless along any axes you want to scroll by

An example of such an image:

![Distortion Map](http://i.imgur.com/2OXLwC3.png)

This image contains coherent noise data in all channels, but we will only be using the R and G channels in our shader.

The set of shaders consist of a trivial passthrough vertex shader and a self-explanatory fragment shader.
```glsl
// heat_shader.vs

#version 130

// Simple passthrough vertex shader... Nothing fancy here.
void main()
{
    gl_Position = gl_ProjectionMatrix * gl_ModelViewMatrix * gl_Vertex;
    gl_TexCoord[0] = gl_TextureMatrix[0] * gl_MultiTexCoord0;
    gl_FrontColor = gl_Color;
}
```

```glsl
// heat_shader.fs

#version 130

uniform sampler2D currentTexture; // Our render texture
uniform sampler2D distortionMapTexture; // Our heat distortion map texture

uniform float time; // Time used to scroll the distortion map
uniform float distortionFactor; // Factor used to control severity of the effect
uniform float riseFactor; // Factor used to control how fast air rises

void main()
{
    vec2 distortionMapCoordinate = gl_TexCoord[0].st;
    
    // We use the time value to scroll our distortion texture upwards
    // Since we enabled texture repeating, OpenGL takes care of
    // coordinates that lie outside of [0, 1] by discarding
    // the integer part and keeping the fractional part
    // Basically performing a "floating point modulo 1"
    // 1.1 = 0.1, 2.4 = 0.4, 10.3 = 0.3 etc.
    distortionMapCoordinate.t -= time * riseFactor;
    
    vec4 distortionMapValue = texture2D(distortionMapTexture, distortionMapCoordinate);

    // The values are normalized by OpenGL to lie in the range [0, 1]
    // We want negative offsets too, so we subtract 0.5 and multiply by 2
    // We end up with values in the range [-1, 1]
    vec2 distortionPositionOffset = distortionMapValue.xy;
    distortionPositionOffset -= vec2(0.5f, 0.5f);
    distortionPositionOffset *= 2.f;

    // The factor scales the offset and thus controls the severity
    distortionPositionOffset *= distortionFactor;

    // The latter 2 channels of the texture are unused... be creative
    vec2 distortionUnused = distortionMapValue.zw;

    // Since we all know that hot air rises and cools,
    // the effect loses its severity the higher up we get
    // We use the t (a.k.a. y) texture coordinate of the original texture
    // to tell us how "high up" we are and damp accordingly
    // Remember, OpenGL 0 is at the bottom
    distortionPositionOffset *= (1.f - gl_TexCoord[0].t);
    
    vec2 distortedTextureCoordinate = gl_TexCoord[0].st + distortionPositionOffset;

    gl_FragColor = gl_Color * texture2D(currentTexture, distortedTextureCoordinate);
}
```

If you want to generate the distortion image procedurally, you will need to get [libnoise](http://libnoise.sourceforge.net/) and uncomment the marked code segment.
```cpp
#include <SFML/Graphics.hpp>

/*
// You will need libnoise to generate the distortion image
#include <noise/noise.h>
#include <noise/noiseutils.h>

void createDistortionMap()
{
    // Increasing the size of the image will enable finer detail
    // This isn't always necessary for coarser distortions
    // since OpenGL interpolates nicely for us anyway
    // in the case where the object is larger than our image
    int width = 256;
    int height = 256;

    module::Perlin perlin;

    // Adjust these values according to your liking
    perlin.SetOctaveCount(4);
    perlin.SetFrequency(4.f / 1.f);
    perlin.SetPersistence(1.f / 8.f);

    // Obviously we only need the R and G channels,
    // but you can use the B and A channels for other effects
    utils::NoiseMap noise_map_r;
    utils::NoiseMap noise_map_g;
    utils::NoiseMap noise_map_b;
    utils::NoiseMap noise_map_a;

    utils::NoiseMapBuilderPlane plane_builder;
    plane_builder.SetSourceModule(perlin);
    plane_builder.SetBounds(0.0, 3.0, 0.0, 3.0);
    plane_builder.SetDestSize(width, height);

    // Enabling seamless is important for being able to scroll the texture
    plane_builder.EnableSeamless(true);

    perlin.SetSeed(2);
    plane_builder.SetDestNoiseMap(noise_map_r);
    plane_builder.Build();

    perlin.SetSeed( 3 );
    plane_builder.SetDestNoiseMap(noise_map_g);
    plane_builder.Build();

    perlin.SetSeed(5);
    plane_builder.SetDestNoiseMap(noise_map_b);
    plane_builder.Build();

    perlin.SetSeed(7);
    plane_builder.SetDestNoiseMap(noise_map_a);
    plane_builder.Build();

    sf::Image image;
    image.create(width, height);

    for (int x = 0; x < width; ++x) {
        for (int y = 0; y < height; ++y) {
            float normalized_r = (noise_map_r.GetValue(x, y) + 1.f) / 2.f;
            float normalized_g = (noise_map_g.GetValue(x, y) + 1.f) / 2.f;
            float normalized_b = (noise_map_b.GetValue(x, y) + 1.f) / 2.f;
            float normalized_a = (noise_map_a.GetValue(x, y) + 1.f) / 2.f;

            sf::Uint8 value_r = static_cast<sf::Uint8>(std::min(std::max(std::floor(normalized_r * 255.f), 0.f), 255.f));
            sf::Uint8 value_g = static_cast<sf::Uint8>(std::min(std::max(std::floor(normalized_g * 255.f), 0.f), 255.f));
            sf::Uint8 value_b = static_cast<sf::Uint8>(std::min(std::max(std::floor(normalized_b * 255.f), 0.f), 255.f));
            sf::Uint8 value_a = static_cast<sf::Uint8>(std::min(std::max(std::floor(normalized_a * 255.f), 0.f), 255.f));

            image.setPixel(x, y, sf::Color(value_r, value_g, value_b, value_a));
        }
    }

    image.saveToFile("distortion_map.png");
}
*/

int main()
{
    //createDistortionMap();

    sf::RenderWindow window(sf::VideoMode(600, 600), "Heat");

    sf::Texture objectTexture;

    if (!objectTexture.loadFromFile("object.jpg"))
    {
        sf::err() << "Failed to load object texture, exiting..." << std::endl;
        return -1;
    }

    sf::Sprite object;
    object.setTexture(objectTexture);
    object.setScale(.5f, .5f);

    sf::Texture distortionMap;

    // It is important to set repeated to true to enable scrolling upwards
    distortionMap.setRepeated(true);

    // Setting smooth to true lets us use small maps even on larger images
    distortionMap.setSmooth(true);

    if (!distortionMap.loadFromFile("distortion_map.png"))
    {
        sf::err() << "Failed to load distortion map, exiting..." << std::endl;
        return -1;
    }

    sf::RenderTexture renderTexture;
    renderTexture.create(400, 300);

    sf::Sprite sprite;
    sprite.setTexture(renderTexture.getTexture());
    sprite.setPosition(100, 150);

    sf::Shader shader;

    if (!shader.loadFromFile("heat_shader.vs", "heat_shader.fs"))
    {
        sf::err() << "Failed to load shader, exiting..." << std::endl;
        return -1;
    }

    shader.setParameter("currentTexture", sf::Shader::CurrentTexture);
    shader.setParameter("distortionMapTexture", distortionMap);

    float distortionFactor = .05f;
    float riseFactor = .2f;

    sf::Clock clock;

    while (true)
    {
        sf::Event event;
        if (window.pollEvent(event)) {
            switch (event.type)
            {
                case sf::Event::Closed: return 0;
                case sf::Event::KeyPressed:
                {
                    switch (event.key.code)
                    {
                        case sf::Keyboard::Escape: return 0;
                        case sf::Keyboard::Add: distortionFactor *= 2.f; break;
                        case sf::Keyboard::Subtract: distortionFactor /= 2.f; break;
                        case sf::Keyboard::Multiply: riseFactor *= 2.f; break;
                        case sf::Keyboard::Divide: riseFactor /= 2.f; break;
                        default: break;
                    }
                    break;
                }
                default: break;
            }
        }

        shader.setParameter("time", clock.getElapsedTime().asSeconds());
        shader.setParameter("distortionFactor", distortionFactor);
        shader.setParameter("riseFactor", riseFactor);

        renderTexture.clear();
        renderTexture.draw(object);
        renderTexture.display();

        window.clear();
        window.draw(sprite, &shader);
        window.display();
    }
}
```