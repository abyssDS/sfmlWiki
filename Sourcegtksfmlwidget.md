# SFMLWidget
## Overview
This class is used to integrate a SFML RenderWindow into a gtkmm userinterface. Just use it like a normal RenderWindow but place it as you would a gtkmm widget.

## Example
Just a simple example of a gtkmm interface that has a SFMLWidget and then a Gtk::Button below it.

```cpp
#include <gtkmm.h>
#include "SFMLWidget.h"
 
int main(int argc, char* argv[])
{
    Gtk::Main kit(argc, argv); //Initialize Gtk
 
    Gtk::Window window; //The GTK window will be our top level Window
 
    //Our RenderWindow will never be below  640x480 (unless we explicitly change it) 
    //but it may be more then that
    SFMLWidget ourRenderWindow(sf::VideoMode(640, 480)); 
 
    // Doesn't draw the renderWindow but makes it so it will draw when we add it to the window
    ourRenderWindow.show();
 
    //VBox is a vertical box, we're going to pack our render window and a button in here
    Gtk::VBox ourVBox;
 
    Gtk::Button ourButton("Hello I do nothing"); //Just a clickable button, it won't be doing anything
    ourButton.show();
 
    ourVBox.pack_start(ourRenderWindow); //Add ourRenderWindow to the top of the VBox
 
    //PACK_SHRINK makes the VBox only allocate enough space to show the button and nothing more
    ourVBox.pack_start(ourButton, Gtk::PACK_SHRINK);
    ourVBox.show();
 
    window.add(ourVBox); //Adds ourVBox to the window so it (and it's children) can be drawn
 
    Gtk::Main::run(window); //Draw the window
    return 0;
}
```

## Source Code
*Note: This was compiled using G++ and gtkmm-2.4 on a machine running Linux Mint 12.04*

### SFMLWidget.h

```cpp
#ifndef SFMLWIDGET_H_INCLUDED
#define SFMLWIDGET_H_INCLUDED

#include <SFML/Graphics.hpp>
#include <gdkmm.h>
#include <gtkmm/widget.h>

class SFMLWidget : public Gtk::Widget, public sf::RenderWindow
{
protected:
    sf::VideoMode m_vMode;

    virtual void on_size_request(Gtk::Requisition* requisition);
    virtual void on_size_allocate(Gtk::Allocation& allocation);
    virtual void on_map();
    virtual void on_unmap();
    virtual void on_realize();
    virtual void on_unrealize();

    Glib::RefPtr<Gdk::Window> m_refGdkWindow;
public:
    SFMLWidget(sf::VideoMode Mode);
    virtual ~SFMLWidget();

    void invalidate();
    void display();
};

#endif
```

### SFMLWidget.cpp

```cpp
#include "SFMLWidget.h"


// Tested on Linux Mint 12.4 and Windows 7
#if defined(SFML_SYSTEM_WINDOWS)

#include <gdk/gdkwin32.h>
#define GET_WINDOW_HANDLE_FROM_GDK GDK_WINDOW_HANDLE

#elif defined(SFML_SYSTEM_LINUX) || defined(SFML_SYSTEM_FREEBSD)

#include <gdk/gdkx.h>
#define GET_WINDOW_HANDLE_FROM_GDK GDK_WINDOW_XID

#elif defined(SFML_SYSTEM_MACOS)

#error Note: You have to figure out an analogue way to access the handle of the widget on a Mac-System

#else

#error Unsupported Operating System

#endif


SFMLWidget::SFMLWidget(sf::VideoMode Mode)
    : sf::RenderWindow(Mode, "")
{
    set_flags(Gtk::NO_WINDOW); // Makes this behave like an interal object rather then a parent window.
}

SFMLWidget::~SFMLWidget()
{
}

void SFMLWidget::on_size_request(Gtk::Requisition* requisition)
{
    *requisition = Gtk::Requisition();

    sf::Vector2u size = this->getSize();

    requisition->width = size.x;
    requisition->height = size.y;
}

void SFMLWidget::on_size_allocate(Gtk::Allocation& allocation)
{
    //Do something with the space that we have actually been given:
    //(We will not be given heights or widths less than we have requested, though
    //we might get more)

    this->set_allocation(allocation);

    if(m_refGdkWindow)
    {
        m_refGdkWindow->move_resize(allocation.get_x(),
                                    allocation.get_y(),
                                    allocation.get_width(),
                                    allocation.get_height() );
        this->setSize(sf::Vector2u(allocation.get_width(),
                                   allocation.get_height()));
    }
}

void SFMLWidget::on_map()
{
    Gtk::Widget::on_map();
}

void SFMLWidget::on_unmap()
{
    Gtk::Widget::on_unmap();
}

void SFMLWidget::on_realize()
{
    Gtk::Widget::on_realize();

    if(!m_refGdkWindow)
    {
        //Create the GdkWindow:
        GdkWindowAttr attributes;
        memset(&attributes, 0, sizeof(attributes));

        Gtk::Allocation allocation = get_allocation();

        //Set initial position and size of the Gdk::Window:
        attributes.x = allocation.get_x();
        attributes.y = allocation.get_y();
        attributes.width = allocation.get_width();
        attributes.height = allocation.get_height();

        attributes.event_mask = get_events () | Gdk::EXPOSURE_MASK;
        attributes.window_type = GDK_WINDOW_CHILD;
        attributes.wclass = GDK_INPUT_OUTPUT;


        m_refGdkWindow = Gdk::Window::create(get_window(), &attributes,
                GDK_WA_X | GDK_WA_Y);
        unset_flags(Gtk::NO_WINDOW);
        set_window(m_refGdkWindow);

        //set colors
        modify_bg(Gtk::STATE_NORMAL , Gdk::Color("red"));
        modify_fg(Gtk::STATE_NORMAL , Gdk::Color("blue"));

        // transparent background
        this->get_window()->set_back_pixmap(Glib::RefPtr<Gdk::Pixmap>());

        this->set_double_buffered(false);

        //make the widget receive expose events
        m_refGdkWindow->set_user_data(gobj());

        this->sf::RenderWindow::create(GET_WINDOW_HANDLE_FROM_GDK(m_refGdkWindow->gobj()));
    }
}

void SFMLWidget::on_unrealize()
{
  m_refGdkWindow.clear();

  //Call base class:
  Gtk::Widget::on_unrealize();
}

void SFMLWidget::display()
{
    if(m_refGdkWindow)
    {
        sf::RenderWindow::display();
    }
}

void SFMLWidget::invalidate()
{
    if(m_refGdkWindow)
    {
        m_refGdkWindow->invalidate(true);
    }
}
```

## Customizing
### Render
In order to render something, you should overload the `on_expose_event` method or connect to the `signal_expose_event()` signal. Don't forget to call the `display()` method. Use `SFMLWidget::display()` instead of `sf::RenderWindow::display()` as it will check whether the Widget has been already initialized.

### Animations
In order to add animate something, call `invalidate()` often enough. This will cause gtkmm to redraw the Widget.

### Example
The following example shows one way to animate and render a circle.

*Note: This was compiled using G++ and gtkmm-2.4 on a machine running Linux Mint 12.04*

```cpp
#include <gtkmm.h>
#include "SFMLWidget.h"

class MovingCircle
{
public:
    // A simple circle shape that will be animated and drawn
    sf::CircleShape circle;
    
    // a reference to our SFMLWidget
    SFMLWidget& widget;
    
    // The radius of our circle
    float radius;
    
    MovingCircle(SFMLWidget& widget) : widget(widget)
    {
        // Set the radius to an unmiportand value
        radius = 32.f;
        circle.setRadius(radius);
        
        // move the circoe to it's first position
        moveToStartPoint();
        
        // Let the animate method be called every 50ms
        // Note: MovingCircle::animate() doesn't return any value, but signal_timeout() expects
        //       a boolean value.
        //       Using sigc::bind_return(true, ...) we get a signal returning always true.
        Glib::signal_timeout().connect(sigc::bind_return(
                                          sigc::mem_fun(this, &MovingCircle::animate),
                                          true),
                                       50);
        
        // Makes our draw Method beeing drawn everytime the widget itself gets drawn.
        // Note: MovingCircle::draw() doesn't accept any parameter, but signal_expose_event() gives one.
        //       Using sigc::hide(...) we get a signal expecting one.
        widget.signal_expose_event().connect(sigc::bind_return(
                                                sigc::hide(
                                                    sigc::mem_fun(this, &MovingCircle::draw)),
                                                true));
                                
        // Everytime the widget gets resized, we need to adjust the view.                
        widget.signal_size_allocate().connect(sigc::hide(
                                                    sigc::mem_fun(this, &MovingCircle::resize_view)));
    }
    
    void animate()
    {
        // Simply move the circle...
        sf::Vector2f position = circle.getPosition();
        position.x += 8.f;
        position.y += 8.f;
        
        // until it "leaves" the Widget
        if(position.x > widget.getSize().x+radius || position.y > widget.getSize().y+radius)
          moveToStartPoint();
        else
          circle.setPosition(position);
          
        // Tell gtkmm that the SFML Widget wants to be redrawn
        widget.invalidate();
    }
    
    void draw()
    {    
        widget.clear();
    
        // SFMLWidget oferrides sf::RenderWindow, so you can use every method of sf::RenderWindow
        widget.draw(circle);
        
        // Calls SFMLWidget::display, whitch checks wether the widget is realized
        // and if so, sf::RenderWindow::display gets called.
        widget.display();
    }
    
    void resize_view()
    {
        // Let the View fit the pixels of the window.
        sf::Vector2f lower_right(widget.getSize().x,
                                 widget.getSize().y);
    
        sf::View view(lower_right * 0.5f,
                      lower_right);
        widget.setView(view);
    }
    
    void moveToStartPoint()
    {
        circle.setPosition(-radius, -radius);
    }
};

int main(int argc, char* argv[])
{
    Gtk::Main kit(argc, argv); //Initialize Gtk
 
    Gtk::Window window; //The GTK window will be our top level Window
 
    //Our RenderWindow will never be below  640x480 (unless we explicitly change it) 
    //but it may be more then that
    SFMLWidget ourRenderWindow(sf::VideoMode(640, 480)); 
    
    MovingCircle moving_circle(ourRenderWindow);
 
    // Doesn't draw the renderWindow but makes it so it will draw when we add it to the window
    ourRenderWindow.show();
 
    //VBox is a vertical box, we're going to pack our render window and a button in here
    Gtk::VBox ourVBox;
 
    Gtk::Button ourButton("Restart"); //Just a clickable button, it won't be doing anything
    ourButton.show();
    ourButton.signal_clicked().connect(sigc::mem_fun(&moving_circle, &MovingCircle::moveToStartPoint));
 
    ourVBox.pack_start(ourRenderWindow); //Add ourRenderWindow to the top of the VBox
 
    //PACK_SHRINK makes the VBox only allocate enough space to show the button and nothing more
    ourVBox.pack_start(ourButton, Gtk::PACK_SHRINK);
    ourVBox.show();
 
    window.add(ourVBox); //Adds ourVBox to the window so it (and it's children) can be drawn
 
    Gtk::Main::run(window); //Draw the window
    return 0;
}
```


## References
* [gtkmm Websit](http://www.gtkmm.org)
* [sf::RenderWindow Documentation](http://www.sfml-dev.org/documentation/2.0/classsf_1_1RenderWindow.php)
* [Gtk::Widget Documentation](http://www.gtkmm.org/docs/gtkmm-2.4/docs/reference/html/classGtk_1_1Widget.html)