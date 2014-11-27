This is a small example the archieve a "letterbox effect" (if I'm using that term correctly) when the window is resized; black bars will appear horizontally or vertically if the aspect ratio of the window is not the same as the game view, and the game view will always stretch keeping the aspect ratio.

```cpp
#include <SFML/Graphics.hpp>
#include <string>
#include <iostream>

int main() {

    // Create a window
    int resX = 640;
    int resY = 480;
    sf::RenderWindow window;
    window.create( sf::VideoMode(resX, resY), "Letterbox", (sf::Style::Resize + sf::Style::Close) );

    // Create a view. In this example the view will be the same size as the window, but it can be any size.
    sf::View view;
    view.setSize( resX, resY );
    view.setCenter( view.getSize().x / 2, view.getSize().y / 2 );

    // Create a sprite. This sprite represents the whole map of a game in this example,
    // so it should be bigger than the view/window, so you can't see all of it on screen.
    sf::Texture texture;
    texture.loadFromFile("map.png");
    sf::Sprite sprite(texture);

    // Game loop.
    while ( window.isOpen() ) {

        // Poll events.
        sf::Event event;
        while ( window.pollEvent(event) ) {

            if (sf::Keyboard::isKeyPressed(sf::Keyboard::Escape) )
                event.type = sf::Event::Closed;

            if (event.type == sf::Event::Closed)
                window.close();

            if (event.type == sf::Event::Resized) {

                // If the window is resized, the following code compares the aspect ratio of the window
                // to the aspect ratio of the view, and sets the viewport accordingly.

                float visibleAreaRatio = event.size.width / (float) event.size.height;
                float viewRatio = view.getSize().x / (float) view.getSize().y;
                float sizeX = 1;
                float sizeY = 1;
                float posX = 0;
                float posY = 0;

                bool horizontalSpacing = true;
                if (visibleAreaRatio < viewRatio)
                    horizontalSpacing = false;

                // If horizontalSpacing is true, the black bars will appear on the left and right side.
                // Otherwise, the black bars will appear on the top and bottom.

                if (horizontalSpacing) {
                    sizeX = viewRatio / visibleAreaRatio;
                    posX = (1 - sizeX) / 2.0;
                }

                else {
                    sizeY = visibleAreaRatio / viewRatio;
                    posY = (1 - sizeY) / 2.0;
                }

                view.setViewport( sf::FloatRect(posX, posY, sizeX, sizeY) );
            }
        }

        // Draw everything.
        window.clear();

        window.setView( view );
        window.draw( sprite );

        window.display();

    }

    return 1;
}
```
