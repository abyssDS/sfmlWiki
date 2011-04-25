# Masking using Sprites, Shapes or Strings

This page shows a way to use Sprites, Shapes or Strings as masks for other sprites. A mask partially or totally obscures another image, and can be used to create effects such as non-rectangular windows, and many others.

## Disclaimer  
This code is targeted for the SFML 1.6 release code, and may not work on SFML 2.0 up.  
It requires changing a line in the original 1.6 source code and recompiling it, as well as adding new types of Sprite, Shapes and Strings, as well as extending the RenderWindow.  
  
Below, I provide the source for each new class, and examples for the usage of each.  
  
**NOTE**: it hasn't been extensively tested so there may be the occasional bug.  

## How it works
By setting a SpriteMasked/ShapeMasked/StringMasked MaskType as "Mask" and drawing it, it will only write that drawable's alpha information (transparency) to the screen, ignoring the sprite image's colors.  
  
By setting the MaskType of another SpriteMasked/ShapeMasked/StringMasked as "BlendWithPreviousMask" and drawing it over an area that has some masking information (a mask previously drawn there), the mask will obscure the second drawn image.  
  
Note: after masking an area, that mask will remain until the screen is cleared. Further writing to this area will still use the original mask. You can clear the mask by using the new "RenderWindowMasked::ClearMask()" function, which clears just the screen's mask information (alpha channel).
  
Check the examples and details below for more on MaskTypes.

## How to install
First: we will have to make the function <code>sf::Drawable::Draw</code> a virtual function, so that we can override it in our new classes.  
Go to the SFML source root folder, and open "include/SFML/Graphics/Drawable.hpp".  
  
Then find the function definition
```cpp
void Draw(RenderTarget& Target) const;
```
(around line 335)
and replace it with
```cpp
virtual void Draw(RenderTarget& Target) const;
```
(add "virtual" at the start).
And recompile the "sfml-graphics" or "sfml" lib.

Now, add these new files to your source:

### New files
  * **MaskTypes.h** - has the several masking types that you can use. Define a MaskType by using "MaskTypes::Type" and set them using "MaskTypes::BlendWithPreviousMask". The mask types are the following:
    * __MaskType::None__ - use the normal sf::BlendMode.
    * __MaskType::Mask__ - use this Drawable's Alpha mask as a Mask (only drawing the Drawable's Alpha info, not any color). Will blend the mask with previous existing masks.
    * __MaskType::MaskOverWritePreviousMask__ - similar to the previous, but will overwrite the previous mask instead of blending into it (allows to "cut" into an existing mask, and is used by example with "StringMask" - check the example)
    * __MaskType::BlendWithPreviousMask__ - set this mask to have a Drawable blend with the mask previously drawn on its area (only draw where alpha > 0)
    * __MaskType::BlendWithPreviousMaskReversed__ - set this mask to have a Drawable blend with the NEGATIVE of the mask previously drawn on its area (transparent areas become opaque and vice-versa).
  * **MaskUtils.h** - Internal class, no need to call this from your code. Basically where all the magic happens, OpenGL calls are made, states are saved and restored, etc. Some code that was in "sf::Drawable" related to blend modes was pasted here.
  * **RenderWindowMask.h** - must be used instead of a regular sf::RenderWindow, inherits from it. Adds the "ClearMask()" function, which clears just the color buffer's alpha channel.
  * **SpriteMasked.h** - use this to draw a Sprite using masking, or to use a Sprite AS a mask. Inherits from sf::Sprite. Adds the "maskSetType/maskGetType" functions necessary to define the masking type.
  * **ShapeMasked.h** - use this to use a Shape as a mask or to mask a Shape. Inherits from sf::Shape. Adds the "maskSetType/maskGetType" functions necessary to define the masking type.
  * **StringMasked.h** - use this to use a String as a mask or to mask a String. Inherits from sf::String. Adds the "maskSetType/maskGetType" functions necessary to define the masking type.

P.S. no need to preserve states (use "App.PreserveOpenGLStates(true)"), it's all done internally.


You will have to use both the new SpriteMasked/ShapeMasked/StringMasked and RenderWindowMasked, instead of the original SFML ones (e.g. you can't attempt to mask a sf::Sprite with a SpriteMasked, both the mask and the masked sprites must be SpriteMasked, with the correct mask types set).

## Source Code
### MaskTypes.h
```cpp
#ifndef MASKTYPES_H_INCLUDED
#define MASKTYPES_H_INCLUDED

class MaskTypes
{
    public:
    enum Type
    {
        None = 0,
        Mask,
        MaskOverWritePreviousMask,
        BlendWithPreviousMask,
        BlendWithPreviousMaskReversed
    };
};

#endif // MASKTYPES_H_INCLUDED
```


### MaskUtils.h
```cpp
#ifndef MASKUTILS_H_INCLUDED
#define MASKUTILS_H_INCLUDED

#include <SFML/Graphics/Drawable.hpp>
#include "MaskTypes.h"

using namespace sf;

class MaskUtils
{
    public:
        MaskUtils()
        {
            reset();
        }

        void reset()
        {
            restoreColorBuffer = false;
            restoreBlendMode = false;
        }

        void setBlendModesBasedOnMaskType(MaskTypes::Type maskType, Blend::Mode blendMode)
        {
            switch(maskType)
            {
                case MaskTypes::None:
                {
                    restoreBlendMode = true;
                    blendModePreviouslyEnabled = glIsEnabled(GL_BLEND);

                    // Setup alpha-blending
                    Blend::Mode myBlendMode = blendMode;
                    if (myBlendMode == Blend::None)
                    {
                        glDisable(GL_BLEND);
                    }
                    else
                    {
                        glEnable(GL_BLEND);

                        switch (myBlendMode)
                        {
                            case Blend::Alpha :    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA); break;
                            case Blend::Add :      glBlendFunc(GL_SRC_ALPHA, GL_ONE);                 break;
                            case Blend::Multiply : glBlendFunc(GL_DST_COLOR, GL_ZERO);                break;
                            default :                                                                 break;
                        }
                    }
                }
                break;

                case MaskTypes::Mask:
                {
                    blendModePreviouslyEnabled = glIsEnabled(GL_BLEND);
                    glEnable(GL_BLEND);
                    glColorMask(false, false, false, true);// we can just write the alpha info
                    glBlendFunc(GL_DST_ALPHA, GL_SRC_ALPHA); // include the source (picture we are drawing) alpha, ignore the destination (info already in color buffer) alpha
                    restoreColorBuffer = true;
                    restoreBlendMode = true;
                }
                break;
                case MaskTypes::MaskOverWritePreviousMask:
                {
                    blendModePreviouslyEnabled = glIsEnabled(GL_BLEND);
                    glEnable(GL_BLEND);
                    glColorMask(false, false, false, true);// we can just write the alpha info
                    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA); // include the source (picture we are drawing) alpha, ignore the destination (info already in color buffer) alpha
                    restoreColorBuffer = true;
                    restoreBlendMode = true;
                }
                break;
                case MaskTypes::BlendWithPreviousMask:
                {
                    blendModePreviouslyEnabled = glIsEnabled(GL_BLEND);

                    // draw stuff inside window
                    glEnable(GL_BLEND);
                    glBlendFunc(GL_ONE_MINUS_DST_ALPHA, GL_DST_ALPHA); // blend with alpha info in color buffer, which was written by previous image
                    restoreBlendMode = true;
                }
                break;
                case MaskTypes::BlendWithPreviousMaskReversed:
                {
                    blendModePreviouslyEnabled = glIsEnabled(GL_BLEND);

                    // draw stuff inside window
                    glEnable(GL_BLEND);
                    glBlendFunc(GL_DST_ALPHA, GL_ONE_MINUS_DST_ALPHA); // blend with alpha info in color buffer, which was written by previous image
                    restoreBlendMode = true;
                }
                break;
            }
        }

        void restoreStates()
        {
            if(restoreColorBuffer)
                glColorMask(true, true, true, true);  // make sure that for further images that we are writing all colors

            if(restoreBlendMode)
            {
                if(blendModePreviouslyEnabled)
                    glEnable(GL_BLEND);
                else
                    glDisable(GL_BLEND);
            }
        }

    private:
        bool restoreColorBuffer;
        bool restoreBlendMode;
        bool blendModePreviouslyEnabled;
};

#endif // MASKUTILS_H_INCLUDED
```


### RenderWindowMask.h
```cpp
#ifndef RENDERWINDOWMASK_H_INCLUDED
#define RENDERWINDOWMASK_H_INCLUDED

#include <SFML/Graphics.hpp>
#include "MaskTypes.h"

using namespace sf;

class RenderWindowMask : public RenderWindow
{
    public:
        RenderWindowMask(
                            VideoMode Mode,
                            const std::string& Title,
                            unsigned long WindowStyle = Style::Resize | Style::Close,
                            const WindowSettings& Params = WindowSettings())
        {
            Create(Mode, Title, WindowStyle, Params);
        }

        RenderWindowMask(   WindowHandle Handle,
                            const WindowSettings& Params = WindowSettings())
        {
            Create(Handle, Params);
        }

        // clears the alpha channel
        void ClearMask()
        {
            glColorMask(false, false, false, true); // disable messing with the rgb channels, just change the alpha
            glClear(GL_COLOR_BUFFER_BIT);    // clear the color buffer (just the alpha channel)
            //glClearColor(FillColor.r / 255.f, FillColor.g / 255.f, FillColor.b / 255.f, FillColor.a / 255.f));
            glColorMask(true, true, true, true); // enable messing with the rgb channels, just change the alpha
        }
};

#endif // RENDERWINDOWMASK_H_INCLUDED
```


### SpriteMasked.h
```cpp
#ifndef SPRITEMASKED_H_INCLUDED
#define SPRITEMASKED_H_INCLUDED

#include <SFML/Graphics/Sprite.hpp>
#include "MaskTypes.h"
#include "MaskUtils.h"

using namespace sf;

class SpriteMasked : public Sprite
{
    public:
        SpriteMasked() :
        Sprite(),
        maskType(MaskTypes::None)
        {
        }

        void Draw(RenderTarget& Target) const
        {
            // Save the current modelview matrix and set the new one
            glMatrixMode(GL_MODELVIEW);
            glPushMatrix();
            glMultMatrixf(GetMatrix().Get4x4Elements());

            MaskUtils maskUtils;
            maskUtils.setBlendModesBasedOnMaskType(this->maskType, GetBlendMode());

            // Set color
            Color myColor = GetColor();
            glColor4f(myColor.r / 255.f, myColor.g / 255.f, myColor.b / 255.f, myColor.a / 255.f);

            // Let the derived class render the object geometry
            Render(Target);

            maskUtils.restoreStates();

            // Restore the previous modelview matrix
            glMatrixMode(GL_MODELVIEW);
            glPopMatrix();
        }

        void maskSetType(MaskTypes::Type maskType)
        {
            this->maskType = maskType;
        }

        MaskTypes::Type maskGetType()
        {
            return this->maskType;
        }

    private:
        MaskTypes::Type maskType;

};

#endif // SPRITEMASKED_H_INCLUDED
```


### ShapeMasked.h
```cpp
#ifndef SHAPEMASKED_H_INCLUDED
#define SHAPEMASKED_H_INCLUDED

#include <SFML/Graphics/Shape.hpp>
#include "MaskTypes.h"
#include "MaskUtils.h"

using namespace sf;

class ShapeMasked : public Shape
{
    public:
        ShapeMasked() :
        Shape(),
        maskType(MaskTypes::None)
        {}

        ShapeMasked(sf::Shape& shape) :
        Shape(),
        maskType(MaskTypes::None)
        {
            // copy points
            for(unsigned int idxPoint = 0; idxPoint < shape.GetNbPoints(); ++idxPoint)
            {
                this->AddPoint(shape.GetPointPosition(idxPoint), shape.GetPointColor(idxPoint), shape.GetPointOutlineColor(idxPoint));
            }

            this->SetOutlineWidth(shape.GetOutlineWidth());
            this->SetColor(shape.GetColor());
        }

        void Draw(RenderTarget& Target) const
        {
            // Save the current modelview matrix and set the new one
            glMatrixMode(GL_MODELVIEW);
            glPushMatrix();
            glMultMatrixf(GetMatrix().Get4x4Elements());

            MaskUtils maskUtils;
            maskUtils.setBlendModesBasedOnMaskType(this->maskType, GetBlendMode());

            // Set color
            Color myColor = GetColor();
            glColor4f(myColor.r / 255.f, myColor.g / 255.f, myColor.b / 255.f, myColor.a / 255.f);

            // Let the derived class render the object geometry
            Render(Target);

            maskUtils.restoreStates();

            // Restore the previous modelview matrix
            glMatrixMode(GL_MODELVIEW);
            glPopMatrix();
        }

        void maskSetType(MaskTypes::Type maskType)
        {
            this->maskType = maskType;
        }

    private:
        MaskTypes::Type maskType;
};

#endif // SHAPEMASKED_H_INCLUDED
```


### StringMasked.h
```cpp
#ifndef TEXTMASKED_H_INCLUDED
#define TEXTMASKED_H_INCLUDED

#include <SFML/Graphics.hpp>
#include "MaskTypes.h"
#include "MaskUtils.h"

using namespace sf;

class StringMasked : public sf::String
{
    public:
        StringMasked() :
        String(),
        maskType(MaskTypes::None)
        {}

        StringMasked(sf::Shape& shape) :
        String(),
        maskType(MaskTypes::None)
        {
        }

        void Draw(RenderTarget& Target) const
        {
            // Save the current modelview matrix and set the new one
            glMatrixMode(GL_MODELVIEW);
            glPushMatrix();
            glMultMatrixf(GetMatrix().Get4x4Elements());

            bool restoreColorBuffer = false;
            bool restoreBlendMode = false;
            bool blendModePreviouslyEnabled;

            MaskUtils maskUtils;
            maskUtils.setBlendModesBasedOnMaskType(this->maskType, GetBlendMode());

            // Set color
            Color myColor = GetColor();
            glColor4f(myColor.r / 255.f, myColor.g / 255.f, myColor.b / 255.f, myColor.a / 255.f);

            // Let the derived class render the object geometry
            Render(Target);

            maskUtils.restoreStates();

            // Restore the previous modelview matrix
            glMatrixMode(GL_MODELVIEW);
            glPopMatrix();
        }

        void maskSetType(MaskTypes::Type maskType)
        {
            this->maskType = maskType;
        }

        MaskTypes::Type maskGetType()
        {
            return this->maskType;
        }

    private:
        MaskTypes::Type maskType;
};

#endif // TEXTMASKED_H_INCLUDED
```


## Example programs
The images used are the following:

  * [[http://sites.google.com/site/pjmendesgames/_/rsrc/1284402627183/home/window.tga]]
  * [[http://sites.google.com/site/pjmendesgames/_/rsrc/1284402613838/home/image.tga]]

### SpriteMasked Example
Two examples: on the left, we can see a simple masking example. On the right, a way to do sub-windowing, by combining overlapping masked elements.
[[http://img841.imageshack.us/img841/6908/examplespritemask.png?400x300]]

```cpp
#include <SFML/System.hpp>
#include <SFML/Graphics.hpp>
#include <iostream>
#include <fstream>

#include "SpriteMasked.h"
#include "RenderWindowMask.h"

int main()
{
    //sf::RenderWindow App(sf::VideoMode(800, 600, 32), "SFML Graphics");
    RenderWindowMask App(sf::VideoMode(800, 600, 32), "SFML Graphics");
    //App.PreserveOpenGLStates(true);

    sf::Clock Clock;

    sf::Image ImageMask;
    ImageMask.LoadFromFile("images/window.tga");

    SpriteMasked spriteMask;
    spriteMask.SetImage(ImageMask);
    spriteMask.SetPosition(200, 200);
    spriteMask.maskSetType(MaskTypes::Mask);


    sf::Image ImageInsideWindow;
    ImageInsideWindow.LoadFromFile("images/image.tga");
    SpriteMasked spriteInsideWindow;
    spriteInsideWindow.SetImage(ImageInsideWindow);
    spriteInsideWindow.SetPosition(200, 200);


    const sf::Input& Input = App.GetInput();
    int             mouseXPrev = Input.GetMouseX(),
                    mouseYPrev = Input.GetMouseY(),
                    mouseX, mouseY;

    float timeSinceLastUpdate;

    // Start game loop
    while (App.IsOpened())
    {
        // Process events
        sf::Event Event;
        while (App.GetEvent(Event))
        {
            // Close window : exit
            if (Event.Type == sf::Event::Closed)
                App.Close();
            if ((Event.Type == sf::Event::KeyPressed) && (Event.Key.Code == sf::Key::Escape))
                App.Close();
        }

        timeSinceLastUpdate = Clock.GetElapsedTime();
        Clock.Reset();

        mouseX = Input.GetMouseX();
        mouseY = Input.GetMouseY();
        float mouseMoveX = (float)(mouseXPrev - mouseX) * timeSinceLastUpdate * 6.0f;
        float mouseMoveY = (float)(mouseYPrev - mouseY) * timeSinceLastUpdate * 6.0f;
        mouseXPrev = mouseX;
        mouseYPrev = mouseY;

        App.Clear(sf::Color(200,0,0,255));


        /////
        ///// Example 1: simple mask
        /////
        // draw sprite to use as mask
        spriteMask.SetScale(1, 1);
        spriteMask.SetPosition(100, 250);
        spriteMask.maskSetType(MaskTypes::Mask);
        App.Draw(spriteMask);

        // draw normal sprite
        spriteInsideWindow.maskSetType(MaskTypes::BlendWithPreviousMask);
        spriteInsideWindow.SetScale(1, 1);
        spriteInsideWindow.SetPosition(100, 250);
        App.Draw(spriteInsideWindow);


        /////
        ///// Example 2: use masking to do sub-windowing
        /////

        // how to draw sub-windows:
        // 1) draw window contents first
        // 2) draw stuff outside afterwards
        // details:
        // 1) draw window contents - background to foreground
        //   a) draw window bg (don't care about mask)
        //   b) draw mask
        //   c) draw "negative" image (BlendWithPreviousMaskReversed)
        //   d) draw other stuff on top of it
        //   e) draw parent mask on top of it
        // 2) draw stuff outside
        //   a) draw "positive" other stuff over it (BlendWithPreviousMask)
        //   ( b) draw mask again if necessary for every new image to draw? dunno)


        // 1) draw window contents - background to foreground
        //   a) draw window bg (don't care about mask)
        //   b) draw mask for next element
        //   c) draw "negative" image (BlendWithPreviousMaskReversed)
        //   d) draw other stuff on top of it
        //   e) draw parent mask on top of it

        //   a) draw window bg (don't care about mask)
        spriteInsideWindow.SetScale(1,1);
        spriteInsideWindow.maskSetType(MaskTypes::None);
        spriteInsideWindow.SetPosition(500, 250);
        App.Draw(spriteInsideWindow);


        //   b) draw mask for next element (1)
        spriteMask.SetScale(0.4, 0.4);
        spriteMask.SetPosition(550, 250);
        spriteMask.maskSetType(MaskTypes::Mask);
        App.Draw(spriteMask);


        //   c) draw "negative" image (BlendWithPreviousMaskReversed) (1)
        spriteInsideWindow.maskSetType(MaskTypes::BlendWithPreviousMask);
        spriteInsideWindow.SetScale(0.4, 0.4);
        spriteInsideWindow.SetPosition(550, 250);
        App.Draw(spriteInsideWindow);


        //   b) draw mask for next element (2)
        spriteMask.SetScale(0.4, 0.4);
        spriteMask.SetPosition(590, 250);
        spriteMask.maskSetType(MaskTypes::Mask);
        App.Draw(spriteMask);

        //   c) draw "negative" image (BlendWithPreviousMaskReversed) (2)
        spriteInsideWindow.maskSetType(MaskTypes::BlendWithPreviousMask);
        spriteInsideWindow.SetScale(0.4, 0.4);
        spriteInsideWindow.SetPosition(590, 250);
        App.Draw(spriteInsideWindow);



        //   e) draw parent mask on top of it
        spriteMask.SetScale(1.0, 1.0);
        spriteMask.SetPosition(500, 200);
        spriteMask.maskSetType(MaskTypes::Mask);
        App.Draw(spriteMask);


        // 2) draw stuff outside
        //   a) draw "positive" other stuff over it (BlendWithPreviousMask)
        spriteInsideWindow.maskSetType(MaskTypes::BlendWithPreviousMaskReversed);
        spriteInsideWindow.SetPosition(350,150);
        spriteInsideWindow.SetScale(2,2);
        App.Draw(spriteInsideWindow);

        // Display window contents on screen
        App.Display();
    }

    return 0;
}
```

### ShapeMasked Example
(masking using an sf::Circle)

[[http://img824.imageshack.us/img824/8628/exampleshapemask.png?400x300]]

```cpp
#include <SFML/System.hpp>
#include <SFML/Graphics.hpp>
#include <iostream>
#include <fstream>

#include "SpriteMasked.h"
#include "ShapeMasked.h"
#include "RenderWindowMask.h"

int main()
{
    //sf::RenderWindow App(sf::VideoMode(800, 600, 32), "SFML Graphics");
    RenderWindowMask App(sf::VideoMode(800, 600, 32), "SFML Graphics");

    sf::Clock Clock;


    sf::Image ImageInsideWindow;
    ImageInsideWindow.LoadFromFile("images/image.tga");
    SpriteMasked spriteInsideWindow;
    spriteInsideWindow.SetImage(ImageInsideWindow);
    spriteInsideWindow.SetPosition(200, 200);
    spriteInsideWindow.maskSetType(MaskTypes::BlendWithPreviousMask);


    const sf::Input& Input = App.GetInput();
    int             mouseXPrev = Input.GetMouseX(),
                    mouseYPrev = Input.GetMouseY(),
                    mouseX, mouseY;

    float timeSinceLastUpdate;


    // masking for shapes
    sf::Shape circle = sf::Shape::Circle(0, 0, 100, sf::Color(255,255,255,0)); // note: zero alpha
    ShapeMasked shapeMask(circle);
    shapeMask.maskSetType(MaskTypes::Mask);

    // Start game loop
    while (App.IsOpened())
    {
        // Process events
        sf::Event Event;
        while (App.GetEvent(Event))
        {
            // Close window : exit
            if (Event.Type == sf::Event::Closed)
                App.Close();
            if ((Event.Type == sf::Event::KeyPressed) && (Event.Key.Code == sf::Key::Escape))
                App.Close();
        }

        timeSinceLastUpdate = Clock.GetElapsedTime();
        Clock.Reset();

        App.Clear(sf::Color(200,0,0,255));


        // draw shape mask
        shapeMask.SetPosition(500,300);
        App.Draw(shapeMask);

        // draw masked image
        spriteInsideWindow.SetPosition(450, 200);
        App.Draw(spriteInsideWindow);


        // Display window contents on screen
        App.Display();
    }

}
```

### StringMasked Example

[[http://img214.imageshack.us/img214/9830/examplestringmask.png?400x300]]

```cpp
#include <SFML/System.hpp>
#include <SFML/Graphics.hpp>
#include <iostream>
#include <fstream>

#include "StringMasked.h"
#include "SpriteMasked.h"
#include "RenderWindowMask.h"


int main()
{
    //sf::RenderWindow App(sf::VideoMode(800, 600, 32), "SFML Graphics");
    RenderWindowMask App(sf::VideoMode(800, 600, 32), "SFML Graphics"); // we'll need to use our RenderWindow to have access to "ClearMask" function
    //App.PreserveOpenGLStates(true);

    //std::ofstream myfile;
    //myfile.open ("pixels.txt");

    sf::Clock Clock;

    sf::Image ImageInsideText;
    ImageInsideText.LoadFromFile("images/image.tga");

    SpriteMasked spriteInsideText;
    spriteInsideText.SetImage(ImageInsideText);

    float timeSinceLastUpdate;


    // masking for text
    // Create a graphical string
    StringMasked stringMask;
    stringMask.SetText("Hello !\nHow are you ?");
    stringMask.SetFont(sf::Font::GetDefaultFont());
    stringMask.SetPosition(100,50);

    stringMask.maskSetType(MaskTypes::Mask);
    stringMask.SetColor(sf::Color(0,0,0,255));

    // Start game loop
    while (App.IsOpened())
    {
        // Process events
        sf::Event Event;
        while (App.GetEvent(Event))
        {
            // Close window : exit
            if (Event.Type == sf::Event::Closed)
                App.Close();
            if ((Event.Type == sf::Event::KeyPressed) && (Event.Key.Code == sf::Key::Escape))
                App.Close();
        }

        timeSinceLastUpdate = Clock.GetElapsedTime();
        Clock.Reset();


        App.Clear(sf::Color(200,0,0,255)); // clear the background to red

        //// How to correctly draw a String to Mask/MaskReverse a sprite:
        //// 1. clear the Window Alpha Mask before drawing the text (or the sprite may appear where you drew previous masks)
        //// 2. draw the SPRITE as the mask (masking everywhere where the sprite would be)
        //// 3. draw the STRING using the mask type "MaskOverWritePreviousMask" (thus "cutting" into the mask the previous sprite left)
        //// 4. draw the Sprite again, normally, using the mask type "BlendWithPreviousMask"

        App.ClearMask(); // 1. clears just the Window's alpha channel

        // 2. use sprite to draw as mask
        spriteInsideText.SetPosition(100,10);

        // save the sprite mask and color state (may be necessary if you reuse this sprite elsewhere)
        MaskTypes::Type prevMaskType = spriteInsideText.maskGetType();
        sf::Color prevSpriteColor = spriteInsideText.GetColor();

        spriteInsideText.SetColor(sf::Color(0,0,0,0));    // set the sprite's alpha to 0
        spriteInsideText.maskSetType(MaskTypes::Mask);    // set the sprite to act as a mask
        App.Draw(spriteInsideText);

        // restore sprite mask+color state
        spriteInsideText.maskSetType(prevMaskType);
        spriteInsideText.SetColor(prevSpriteColor);


        // 3. draw text mask
        stringMask.SetPosition(100,50);
        stringMask.maskSetType(MaskTypes::MaskOverWritePreviousMask);
        App.Draw(stringMask);

        // 4. draw the masked sprite
        spriteInsideText.SetPosition(100,10);
        // using this blending mode, the sprite will be drawn normally and the text area will be "cut" from the sprite
        spriteInsideText.maskSetType(MaskTypes::BlendWithPreviousMask);

        App.Draw(spriteInsideText);




        // Example 2:
        App.ClearMask(); // 1. clears just the Window's alpha channel

        // 2. use sprite to draw as mask
        spriteInsideText.SetPosition(500,10);

        // save the sprite mask and color state (may be necessary if you reuse this sprite elsewhere)
        prevMaskType = spriteInsideText.maskGetType();
        prevSpriteColor = spriteInsideText.GetColor();

        spriteInsideText.SetColor(sf::Color(0,0,0,0));    // set the sprite's alpha to 0
        spriteInsideText.maskSetType(MaskTypes::Mask);    // set the sprite to act as a mask
        App.Draw(spriteInsideText);

        // restore sprite mask+color state
        spriteInsideText.maskSetType(prevMaskType);
        spriteInsideText.SetColor(prevSpriteColor);


        // 3. draw text mask
        stringMask.SetPosition(500,50);
        stringMask.maskSetType(MaskTypes::MaskOverWritePreviousMask);
        App.Draw(stringMask);

        // 4. draw the masked sprite
        spriteInsideText.SetPosition(500,10);

        // using this blending mode, the sprite will be drawn "inside" the text's characters
        spriteInsideText.maskSetType(MaskTypes::BlendWithPreviousMaskReversed);

        App.Draw(spriteInsideText);


        App.Display(); // Display window contents on screen
    }

    return 0;
}
```


This method has a few inconveniences, such as having the overhead of redrawing polygons, and Strings have some boring extra work needed, and work the other way around than sprites and shapes, but I think it's altogether a good and flexible solution, and all-encompassing, technology-wise (supporting older hardware).


## Debugging the alpha channel
Here's a little function for debugging, that saves the screen's current alpha buffer to a file named "imageAlpha.png" (upside down, OpenGL stuff). May help you debug if you run into problems.

```cpp
void GetGLPixels()
{
    unsigned int startX = 0, startY = 0;
    unsigned int width = 800, height = 600;

    GLubyte* pixels = new GLubyte[width * height * 4];

    glReadPixels(startX, startY, width, height, GL_RGBA, GL_UNSIGNED_BYTE, pixels);


    for(unsigned int idxPixelRow = 0; idxPixelRow < height; idxPixelRow++)
    {
        for(unsigned int idxPixelColumn = 0; idxPixelColumn < width; idxPixelColumn++)
        {
            unsigned int index = ((idxPixelRow * width) + idxPixelColumn) * 4;
            unsigned int indexAlpha = ((idxPixelRow * width) + idxPixelColumn);

            float valueRed = pixels[index + 0];
            float valueGreen = pixels[index + 1];
            float valueBlue = pixels[index + 2];
            float valueAlpha = pixels[index + 3];

            pixels[index] = pixels[index + 1] = pixels[index + 2] = 0;
        }
    }


    // create and save image
    sf::Image imageAlpha;
    imageAlpha.LoadFromPixels(width, height, pixels);
    imageAlpha.SaveToFile("imageAlpha.png");


    delete pixels;
}
```

TurboLento aka [[http://pjmendes.blogspot.com|pjmendes]]