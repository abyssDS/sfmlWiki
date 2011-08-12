# Manage different screens in a game

When speaking about screens, I mean Menu screen, Config screen, Game screen, etc... Those screens you find in every games. The problem here is that each screen can be compared to a small SFML application : Each screen will have its own events and will use some variables useless for other screens.

So, we need to separate each screen in order to avoid conflicts. With SFML, it's very simple to do that! You just have to create a cScreen class which will represent each screen. This is an virtual object and it's containing only one method:

```cpp
class cScreen
{
public :
    virtual int Run (sf::RenderWindow &App) = 0;
};
```

This object is made very simple in order to allow great compatibility with every application structure. Run is a method called from the main function, and it returns a positive integer if an other screen must be shown. Otherwise, it returns -1 and the application exits. With a simple object, we have now some screens containing their owns variables and an SFML event loop...

The old SFML Event loop is replaced with :

```cpp
while (screen >= 0)
{
screen = Screens[screen]->Run(App);
}
```

Therefore, you just have to specify the first screen and get into the loop. Each screen will returns, saying the app continues with an other screen or stops. Each Run method get the sf::RenderWindow variable in order to get events and draw sprites, etc...

More, screens are objects created in the main function of the application. So, they keep their values as long as they are alive. It permits to get bak to Config menu while being in game and return to the game without being forced to save everything.

Here is a demo app using the cScreen object :

\- A simple main function

```cpp
#include <fstream>
#include <iostream>
#include <sfml/Graphics.hpp>
#include "screens.hpp"

int main(int argc, char** argv)
{
    //Applications variables
    std::vector<cScreen*> Screens;
    int screen = 0;

    //Window creation
    sf::RenderWindow App(sf::VideoMode(640, 480, 32), "SFML Demo 3");

    //Mouse cursor no more visible
    App.ShowMouseCursor(false);

    //Screens preparations
    screen_0 s0;
    Screens.push_back (&s0);
    screen_1 s1;
    Screens.push_back (&s1);

    //Main loop
    while (screen >= 0)
    {
        screen = Screens[screen]->Run(App);
    }

    return EXIT_SUCCESS;
}
```


\- screens.hpp contains the cScreen include, plus the hpp of each screens of the game. Here, screen_0 is a menu, and screen_1 is the game. Each screen inherits from cScreen :

```cpp
#ifndef SCREENS_HPP_INCLUDED
#define SCREENS_HPP_INCLUDED

//Basic Screen Class
#include "screen.hpp"

//Including each screen of application
#include "screen_0.hpp"
#include "screen_1.hpp"

#endif // SCREENS_HPP_INCLUDED
```


\- screen_0.hpp contains the object definition and its code. "Run" is like the main loop you can have in a simple SFML application. The constructor defines the first value of private members which are finally more like static variables in our case. This permits to have a fade presentation at first loading, since the menu will appear directly. Once background is shown, we use two string to make a menu in which you can navigate with direction's key. Choosing "Play" will go to the game, "Exit" will exit the application by returning -1 to the main function :

```cpp
#include <iostream>
#include "screen.hpp"

class screen_0 : public Screen
{
private:
    int alpha_max;
    int alpha_div;
    bool playing;
public:
    screen_0 (void);
    virtual int Run (sf::RenderWindow &App);
};

screen_0::screen_0 (void)
{
    alpha_max = 3*255;
    alpha_div = 3;
    playing = false;
}

int screen_0::Run (sf::RenderWindow &App)
{
    sf::Event Event;
    bool Running = true;
    sf::Image Image;
    sf::Sprite Sprite;
    int alpha = 0;
    sf::Font Font;
    sf::String Menu1;
    sf::String Menu2;
    sf::String Menu3;
    int menu = 0;

    if (!Image.LoadFromFile("presentation.png"))
    {
        std::cerr<<"Error loading presentation.gif"<<std::endl;
        return (-1);
    }
    Sprite.SetImage(Image);
    Sprite.SetColor(sf::Color(255, 255, 255, alpha));
    if (!Font.LoadFromFile("verdanab.ttf"))
    {
        std::cerr<<"Error loading verdanab.ttf"<<std::endl;
        return (-1);
    }
    Menu1.SetFont(Font);
    Menu1.SetSize(20);
    Menu1.SetText("Play");
    Menu1.SetX(280);
    Menu1.SetY(160);
    Menu2.SetFont(Font);
    Menu2.SetSize(20);
    Menu2.SetText("Exit");
    Menu2.SetX(280);
    Menu2.SetY(220);
    Menu3.SetFont(Font);
    Menu3.SetSize(20);
    Menu3.SetText("Continue");
    Menu3.SetX(280);
    Menu3.SetY(160);

    //Clearing screen
    App.SetBackgroundColor(sf::Color(0, 0, 0, 0));

    if (playing)
    {
        alpha = alpha_max;
    }

    while (Running)
    {
        //Verifying events
        while (App.GetEvent(Event))
        {
            // Window closed
            if (Event.Type == sf::Event::Closed)
            {
                return (-1);
            }
            //Key pressed
            if (Event.Type == sf::Event::KeyPressed)
            {
                switch (Event.Key.Code)
                {
                    case sf::Key::Up:
                        menu = 0;
                        break;
                    case sf::Key::Down:
                        menu = 1;
                        break;
                    case sf::Key::Return:
                        if (menu==0)
                        {
                            //Let's get play !
                            playing = true;
                            return (1);
                        }
                        else
                        {
                            //Let's get work...
                            return (-1);
                        }
                        break;
                    default :
                        break;
                }
            }
        }
        //When getting at alpha_max, we stop modifying the sprite
        if (alpha<alpha_max)
        {
            alpha++;
        }
        Sprite.SetColor(sf::Color(255, 255, 255, alpha/alpha_div));
        if (menu==0)
        {
            Menu1.SetColor(sf::Color(255, 0, 0, 255));
            Menu2.SetColor(sf::Color(255, 255, 255, 255));
            Menu3.SetColor(sf::Color(255, 0, 0, 255));
        }
        else
        {
            Menu1.SetColor(sf::Color(255, 255, 255, 255));
            Menu2.SetColor(sf::Color(255, 0, 0, 255));
            Menu3.SetColor(sf::Color(255, 255, 255, 255));
        }

        //Drawing
        App.Draw(Sprite);
        if (alpha==alpha_max)
        {
            if (playing)
            {
                App.Draw(Menu3);
            }
            else
            {
                App.Draw(Menu1);
            }
            App.Draw(Menu2);
        }
        App.Display();
    }

    //Never reaching this point normally, but just in case, exit the application
    return (-1);
}
```

\- screen_1.hpp contains the game itself. We can't exit from the game directly. If you push Escape, you'll go on the Main Menu. If you move the "player" and go to the menu, you can go back to the game and your position will stay the same. Because your object screen_1 has not been destroyed :

```cpp
#include <iostream>
#include "screen.hpp"

class screen_1 : public Screen
{
private:
    int movement_step;
    int posx;
    int posy;
    sf::Sprite Sprite;
public:
    screen_1 (void);
    virtual int Run (sf::RenderWindow &App);
};

screen_1::screen_1 (void)
{
    movement_step = 5;
    posx = 320;
    posy = 240;
    //Setting sprite
    Sprite.SetColor(sf::Color(255, 255, 255, 150));
    Sprite.SetSubRect(sf::IntRect(0, 0, 10, 10));
}

int screen_1::Run (sf::RenderWindow &App)
{
    sf::Event Event;
    bool Running = true;


    //Clearing screen
    App.SetBackgroundColor(sf::Color(0, 0, 0, 0));

    while (Running)
    {
        //Verifying events
        while (App.GetEvent(Event))
        {
            // Window closed
            if (Event.Type == sf::Event::Closed)
            {
                return (-1);
            }
            //Key pressed
            if (Event.Type == sf::Event::KeyPressed)
            {
                switch (Event.Key.Code)
                {
                    case sf::Key::Escape:
                        return (0);
                        break;
                    case sf::Key::Up:
                        posy -= movement_step;
                        break;
                    case sf::Key::Down:
                        posy += movement_step;
                        break;
                    case sf::Key::Left:
                        posx -= movement_step;
                        break;
                    case sf::Key::Right:
                        posx += movement_step;
                        break;
                    default:
                        break;
                }
            }
        }

        //Updating
        if (posx>630)
            posx = 630;
        if (posx<0)
            posx = 0;
        if (posy>470)
            posy = 470;
        if (posy<0)
            posy = 0;
        Sprite.SetPosition(posx, posy);

        //Drawing
        App.Draw(Sprite);
        App.Display();
    }

    //Never reaching this point normally, but just in case, exit the application
    return -1;
}
```

Here it is, while using this class, you've separated each part of your application not changing very much your app.

Good luck to you now!