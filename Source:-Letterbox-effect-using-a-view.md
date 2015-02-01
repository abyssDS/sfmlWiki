This is a small function to achieve a "letterbox effect" (if I'm using that term correctly) when the window is resized, using a view; black bars will appear horizontally or vertically if the aspect ratio of the window is not the same as the game view, and the game view will always stretch keeping the aspect ratio. 

This is achieved modifying the viewport of the view, so this will only work for views that have a default viewport; it will screw up views with different viewport values (a solution for this may come in the future)

![](http://i.imgur.com/AGVNlMX.png?1)

![](http://i.imgur.com/r86WeBf.png?1)

![](http://i.imgur.com/oO7kbkz.png?1)

##The function

```cpp
sf::View getLetterboxView(sf::View view, int windowWidth, int windowHeight) {

    // Compares the aspect ratio of the window to the aspect ratio of the view,
    // and sets the view's viewport accordingly in order to archieve a letterbox effect.
    // A new view (with a new viewport set) is returned.

    float windowRatio = windowWidth / (float) windowHeight;
    float viewRatio = view.getSize().x / (float) view.getSize().y;
    float sizeX = 1;
    float sizeY = 1;
    float posX = 0;
    float posY = 0;

    bool horizontalSpacing = true;
    if (windowRatio < viewRatio)
        horizontalSpacing = false;

    // If horizontalSpacing is true, the black bars will appear on the left and right side.
    // Otherwise, the black bars will appear on the top and bottom.

    if (horizontalSpacing) {
        sizeX = viewRatio / windowRatio;
        posX = (1 - sizeX) / 2.0;
    }

    else {
        sizeY = windowRatio / viewRatio;
        posY = (1 - sizeY) / 2.0;
    }

    view.setViewport( sf::FloatRect(posX, posY, sizeX, sizeY) );

    return view;
}
```

##Usage example

The function should be used both when you create the view and also when the "Resized" event is triggered in the poll events loop.

If you have more than one view, you need to apply the function to every one of them.

```cpp
#include <SFML/Graphics.hpp>

int main() {

    // Create a window
    int resX = 640;
    int resY = 480;
    sf::RenderWindow window( sf::VideoMode(resX, resY), "Letterbox", (sf::Style::Resize + sf::Style::Close) );

    // Create a view. This can be of any size, but in this example it will be the same size as the window.
    // After creating it, it applies the letterbox effect to it; this is not neccesary if the view
    // is the same size as the window initially, but let's do this just in case.
    sf::View view;
    view.setSize( resX, resY );
    view.setCenter( view.getSize().x / 2, view.getSize().y / 2 );
    view = getLetterboxView( view, resX, resY );  

    // Create a sprite. This sprite represents the whole map of a game in this example.
    sf::Texture texture;
    texture.loadFromFile("map.png");
    sf::Sprite sprite(texture);

    // Game loop.
    while ( window.isOpen() ) {

        // Poll events.
        sf::Event event;
        while ( window.pollEvent(event) ) {

            if (event.type == sf::Event::Resized)
                view = getLetterboxView( view, event.size.width, event.size.height );
        }

        // Draw everything.
        window.clear();

        window.setView(view); 
        window.draw(sprite);

        window.display();
    }

    return 1;
}
```

##Disabling the effect

If you need to disable the effect for some reason, use the following function (which basically just resets the viewport of the view).

```cpp
sf::View disableLetterboxView(sf::View view) {

    view.setViewport( sf::FloatRect(0, 0, 1, 1) );
    return view;
}
```