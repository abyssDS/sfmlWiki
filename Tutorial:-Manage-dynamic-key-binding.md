„Bind“ a key means linking an action to a specific key. For example, you can make the player to use direction keys to move and the Return key to fire. You can also make the left mouse button to shoot in a FPS. In fact, the action is linked to a specific Event.

Ok, that's great, but how can we change all those binds if the player is left-handed? He would probably use the Right mouse button instead of the left one! Binding is making this possible simply. You just have to say that this action is linked to this new Event.

My method is certainly not the best one, but it works and is sufficient for me.

First of all, you have to detect each SFML Event type: mouse, keyboard, joystick. Because each input device is different, you have to treat them separately.

```cpp
enum InputType
{
    KeyboardInput,
    MouseInput,
    JoystickInput
};
```

Then, you must define an action Binding:

```cpp
struct MyKeys
{
    InputType myInputType;
    sf::Event::EventType myEventType;
    sf::Key::Code myKeyCode;
    sf::Mouse::Button myMouseButton;
};
```

Here it is, with this simple structure, you can link an action to a specific input Event;

You have to add, in the main application, the std::map containing the bindings. You can fill it simply or with a xml file, etc… :

```cpp
std::map<std::string,MyKeys> Keys;
MyKeys key;
//Let's bind the left mouse button to the "Shoot" action
key.myInputType = MouseInput;
key.myEventType = sf::Event::MouseButtonPressed;
key.myMouseButton = sf::Mouse::Left;
Keys["Shoot"] = key;
//Let's bind the Return key to the "Jump" action
key.myInputType = KeyboardInput;
key.myEventType = sf::Event::KeyPressed;
key.myKeyCode = sf::Key::Return;
Keys["Jump"] = key;
//Let's bind the Left Control key to the "Use" action
key.myInputType = KeyboardInput;
key.myEventType = sf::Event::KeyPressed;
key.myKeyCode = sf::Key::LControl;
Keys["Use"] = key;
```

Using a std::map permit a simply use of actions directly by their name. We have now to modify our „Freaky“ Event Manager to make it dynamic!

```cpp
//Manage Events
while (App.GetEvent(Event))
{
   bla bla bla...
   //My events
   //Shoot Event
   if (TestEvent(Keys["Shoot"],Event))
   {
      Shoot ();
   }
   if (TestEvent(Keys["Jump"],Event))
   {
      Jump ();
   }
   if (TestEvent(Keys["Use"],Event))
   {
      Use ();
   }
}
```

So, we see that we are managing each action separately with her name. It's now possible to use a function or code directly, or anything else.

The TestEvent function:

```cpp
bool TestEvent (MyKeys k, sf::Event e)
{
    //Mouse event
    if (k.myInputType==MouseInput && k.myEventType==e.Type && k.myMouseButton==e.MouseButton.Button)
    {
        return (true);
    }
    //Keyboard event
    if (k.myInputType==KeyboardInput && k.myEventType==e.Type && k.myKeyCode==e.Key.Code)
    {
        return (true);
    }
    return (false);
}
```

There, each time the function returns „true“, the action can be done. No need to say what exactly has been done, it just has been done!

Here is a complete sample code for testing: 

```cpp
#include <iostream>
#include <fstream>
#include <SFML/Graphics.hpp>
 
enum InputType
{
    KeyboardInput,
    MouseInput,
    JoystickInput
};
 
struct MyKeys
{
    InputType myInputType;
    sf::Event::EventType myEventType;
    sf::Key::Code myKeyCode;
    sf::Mouse::Button myMouseButton;
};
 
bool TestEvent (MyKeys k, sf::Event e);
void Shoot (void);
void Jump(void);
 
int main(int argc, char** argv)
{
    //Variables for main
    sf::RenderWindow App;
    bool Running = true;
    sf::Event Event;
 
    //Variables for demo
    std::map<std::string,MyKeys> Keys;
    MyKeys key;
    //Let's bind the left mouse button to the "Shoot" action
    key.myInputType = MouseInput;
    key.myEventType = sf::Event::MouseButtonPressed;
    key.myMouseButton = sf::Mouse::Left;
    Keys["Shoot"] = key;
    //Let's bind the Return key to the "Jump" action
    key.myInputType = KeyboardInput;
    key.myEventType = sf::Event::KeyPressed;
    key.myKeyCode = sf::Key::Return;
    Keys["Jump"] = key;
    //Let's bind the Left Control key to the "Use" action
    key.myInputType = KeyboardInput;
    key.myEventType = sf::Event::KeyPressed;
    key.myKeyCode = sf::Key::LControl;
    Keys["Use"] = key;
 
    //Window creation
    App.Create(sf::VideoMode(640, 480, 16), "config test");
 
    //Main loop
    while (Running)
    {
        //Manage Events
        while (App.GetEvent(Event))
        {
            //Using Event normally
 
            //Window closed
            if (Event.Type == sf::Event::Closed)
            {
                Running = false;
            }
 
            //Key pressed
            if (Event.Type == sf::Event::KeyPressed)
            {
                switch (Event.Key.Code)
                {
                    case sf::Key::Escape :
                        Running = false;
                        break;
                    case sf::Key::A :
                        std::cout<<"Key A !"<<std::endl;
                        break;
                    default :
                        break;
                }
            }
 
            //Using Event for binding
            //Shoot
            if (TestEvent(Keys["Shoot"],Event))
            {
                //You can use a function
                Shoot ();
            }
            if (TestEvent(Keys["Jump"],Event))
            {
                Jump ();
            }
            if (TestEvent(Keys["Use"],Event))
            {
                //or only code
                std::cout<<"Use !"<<std::endl;
            }
        }
 
        //Display the result
        App.Display();
    }
 
    //End of application
    return EXIT_SUCCESS;
}
 
bool TestEvent (MyKeys k, sf::Event e)
{
    //Mouse event
    if (k.myInputType==MouseInput && k.myEventType==e.Type && k.myMouseButton==e.MouseButton.Button)
    {
        return (true);
    }
    //Keyboard event
    if (k.myInputType==KeyboardInput && k.myEventType==e.Type && k.myKeyCode==e.Key.Code)
    {
        return (true);
    }
    return (false);
}
 
void Shoot (void)
{
    std::cout<<"Shoot !"<<std::endl;
}
 
void Jump (void)
{
    std::cout<<"Jump !"<<std::endl;
}
```