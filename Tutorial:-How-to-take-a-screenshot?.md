Ability to save a screenshot is a nice addition to game applications.

The following function implements application screenshot: it saves to an image file what is currently rendered on a render window.

Use a timestamp-based filename to generate a unique filename for each screenshot.

```cpp
void take_screenshot(const sf::RenderWindow& window, const std::string& filename)
{
    sf::Texture texture;
    texture.create(window.getSize().x, window.getSize().y);
    texture.update(window);
    if (texture.copyToImage().saveToFile(filename))
    {
        std::cout << "screenshot saved to " << filename << std::endl;
    }
    else
    {
        // Error handling
    }
}
```