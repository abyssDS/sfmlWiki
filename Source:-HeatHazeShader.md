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