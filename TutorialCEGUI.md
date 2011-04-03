# Using CEGUI In SFML
[CEGUI](http://cegui.org.uk) is a "free library providing windowing and widgets for graphics APIs / engines where such functionality is not natively available, or severely lacking. The library is object orientated, written in C++, and targeted at games developers who should be spending their time creating great games, not building GUI sub-systems!"  
This will be a tutorial on using CEGUI in SFML.  
We will not get into details on CEGUI. 
For more general CEGUI tutorials, see: <http://www.cegui.org.uk/wiki/index.php>

## Screenshot 
![ ](http://i107.photobucket.com/albums/m300/silentdae/sfmlCEGUI.png?300)  
This is what we are going to create. 
We have a dialog that we can drag around.
Clicking the close button on the dialog will close the entire app. 
The dialog has an editbox where we can type text.
When the text changes, we change an sf::String which is displayed.

## The SDK
Go to <http://cegui.org.uk> and download the SDK. As of this writing, the current stable branch is 0.6.1.  
I downloaded the binary SDK for VC9, CEGUI-SDK-0.6.1-vc9.zip. Now extract it (using 7-zip/unzip/etc).

## Setting Up Your Project
As both SFML and CEGUI are cross-platform, these instructions will be a bit general.

Your compiler will need to know where CEGUIs include files are.
I used a relative path of "..\CEGUI-SDK-0.6.0-vc9\include".
Your linker will need to know where CEGUIs library files are.
For me this was "..\CEGUI-SDK-0.6.0-vc9\lib".
If you are going to use CEGUIs static libraries, you will also need to add "..\CEGUI-SDK-0.6.0-vc9\dependencies\lib".
I will be using static libraries here.
If you are not using static libraries, be sure that your app can access all the shared libraries (dll/so) it requires.
Here is a list of all of the libraries the example (included at the end) was linked with in Release mode:

* sfml-graphics.lib 
* sfml-main.lib 
* sfml-system.lib 
* sfml-window.lib 
* glu32.lib 
* pcre.lib 
* CEGUIBase_Static.lib 
* CEGUIExpatParser_Static.lib 
* expat.lib 
* CEGUISILLYImageCodec_Static.lib 
* SILLY.lib 
* OpenGLGUIRenderer_Static.lib 
* CEGUIFalagardWRBase_Static.lib 

Note that even though most of these are static libraries, I still needed SILLY.dll.
There doesn't seem to be a static version of it.

## Basic Framework 
Yes, we're going to use a little framework.
Don't worry, it's not going to be some huge monster that takes over your app.  
Let's look at the classes:

*  **App** - This is the main class. It takes care of the specifics.
*  **GUIManager** - This is...well, the GUI manager. It's like a tiny wrapper around CEGUI.

Yes, that's all! Now, let's go through each source file.

### Main.cpp

```cpp
#include "App.h"

int main(int argc, char *argv[])
{
	App theApp;

	if (!theApp.Initialize())
	{
		printf("Failed to initialize the app!");
		return 0;
	}
	theApp.Run();
}
```

Easy. We just create an app object, initialize it, and run it.

### App.h

```cpp
#ifndef App_h
#define App_h

#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/Graphics/String.hpp>
#include "GUIManager.h"

class App
{
public:
	static const unsigned int Width = 800, Height = 600;

	App();

	bool Initialize();
	void Run();
	void Stop() {mDone = true;}

	bool onDialog_Closed(const CEGUI::EventArgs& e);
	bool onEditbox_TextChanged(const CEGUI::EventArgs& e);

private:
	sf::RenderWindow mWindow;
	GUIManager mGUIManager;
	bool mDone;
	sf::String mString;
};

#endif
```

There's not much to look at here except this:
```cpp
bool onDialog_Closed(const CEGUI::EventArgs& e);
bool onEditbox_TextChanged(const CEGUI::EventArgs& e);
```

These two functions will be respond to events.
The first one will be called when the close button of our dialog has been clicked.
The second will be called when the text of our Editbox has changed.  
Also...
```cpp
GUIManager mGUIManager;
```

We'll look at it later.  
Let's move on to...

### App.cpp

This is a bigger one. We don't need to look at *all* of it. We'll look at a few key methods.
```cpp
bool App::Initialize()
{
	sf::VideoMode Mode(Width, Height, 32);

	mWindow.Create(Mode, "sfmlCEGUI", sf::Style::Close);
	mWindow.ShowMouseCursor(false);
	if (!mGUIManager.Initialize(&mWindow))
		return false;

	mString.SetPosition(300.0f, 400.0f);
	/***************** Create our dialog **********************/
	try
	{
		CEGUI::WindowManager* WindowMgr = mGUIManager.getWindowManager();
		CEGUI::Window* Dialog = WindowMgr->createWindow("WindowsLook/FrameWindow", "OurDialog");
		Dialog->setMinSize(UVector2(UDim(0.0f, 200),UDim(0.0f, 150)));
		Dialog->setSize(UVector2(UDim(0.0f, 400),UDim(0.0f, 300)));
		Dialog->setPosition(UVector2(UDim(0.25f, 0), UDim(0.1f, 0)));
		Dialog->setText("Window");
		Dialog->subscribeEvent(FrameWindow::EventCloseClicked, Event::Subscriber(&App::onDialog_Closed, this));
		CEGUI::Window* EditBox = WindowMgr->createWindow("WindowsLook/Editbox", "OurDialog_Editbox");
		EditBox->setMinSize(UVector2(UDim(0.0f, 100), UDim(0.0f, 20)));
		EditBox->setSize(UVector2(UDim(0.5f, 0), UDim(0.1f, 0)));
		EditBox->setPosition(UVector2(UDim(0.25f, 0), UDim(0.4f, 0)));
		EditBox->subscribeEvent(CEGUI::Window::EventTextChanged, Event::Subscriber(&App::onEditbox_TextChanged, this));
		Dialog->addChildWindow(EditBox);
		mGUIManager.setRootWindow(Dialog);
	}
	catch (CEGUI::Exception& e)
	{
		printf("CEGUI Exception: %s\n", e.getMessage().c_str());
		return false;
	}
	return true;
}
```

Let's look at a few pieces here:
```cpp
mWindow.ShowMouseCursor(false);
```

CEGUI is going to draw a cursor for us, so we need to tell SFML to hide the system cursor.
```cpp
if (!mGUIManager.Initialize(&mWindow))
	return false;
```

We'll be looking at this method later. It's clear what it does, it simply initializes CEGUI.
```cpp
CEGUI::Window* Dialog = WindowMgr->createWindow("WindowsLook/FrameWindow", "OurDialog");
```

Here we're creating our dialog and giving it the name "OurDialog". This isn't *really* necessary but we do it anyways.
```cpp
CEGUI::Window* EditBox = WindowMgr->createWindow("WindowsLook/Editbox", "OurDialog_Editbox");
```

Then we create our Editbox, giving it the name "OurDialog_Editbox". This *is* necessary, we will use this name in an event handler later.  
```cpp
void App::Run()
{
	sf::Event Event;

	while (!mDone)
	{
		while (mWindow.GetEvent(Event))
		{
			//See if CEGUI should handle it
			if (mGUIManager.handleEvent(Event))
				continue;

			switch (Event.Type)
			{
			case sf::Event::Closed:
				Stop();
				break;
			case sf::Event::KeyPressed:
				if (Event.Key.Code == sf::Key::Escape)
					Stop();

				break;
			}
		}
		mWindow.Draw(mString);
		mGUIManager.Draw();
		mWindow.Display();
	}
}
```

```cpp
if (mGUIManager.handleEvent(Event))
	continue;
```

Here we are checking to see if CEGUI needs to handle this event. If CEGUI handles it, we need to ignore it.
```cpp
mWindow.Draw(mString);
mGUIManager.Draw();
```

First we draw our SFML string. This string is updated to be whatever text is in our Editbox. We'll look at that in a sec.  
We also tell our GUIManager to draw. Simple.
```cpp
/************************************************************************/
/*                            Private                                 * /
/************************************************************************/

bool App::onDialog_Closed(const CEGUI::EventArgs& e)
{
	Stop();
	return true;
}

bool App::onEditbox_TextChanged(const CEGUI::EventArgs& e)
{
	CEGUI::WindowManager* WindowMgr = mGUIManager.getWindowManager();

	CEGUI::Window* Edit = WindowMgr->getWindow("OurDialog_Editbox");
	mString.SetText(Edit->getText().c_str());
	return true;
}
```

Finally, here are our event handlers. The first one, onDialog_Closed, is called when the close button of our dialog is clicked. It calls the App::Stop() method which simply sets mDone to true, thus ending our render loop.  
The second method is called whenever the text in our Editbox changes. We use this event to update our sf::String that we display. Notice that we use the name we gave the Editbox earlier.

### GUIManager.h

```cpp
#ifndef GUIManager_H
#define GUIManager_H

#include <CEGUI.h>
#include <openglrenderer.h>
#include <SFML/Graphics/RenderWindow.hpp>

class GUIManager
{
public:
	GUIManager();
	~GUIManager();

	bool Initialize(sf::RenderWindow* Win);
	bool handleEvent(sf::Event& Event);
	void Draw();

	CEGUI::System* getSystem() {return mSystem;}
	CEGUI::WindowManager* getWindowManager() {return mWindowManager;}
	void setRootWindow(CEGUI::Window* Win) {mSystem->setGUISheet(Win);}

private:
	sf::RenderWindow* mWindow;
	const sf::Input* mInput;

	CEGUI::System* mSystem;
	CEGUI::OpenGLRenderer* mRenderer;
	CEGUI::WindowManager* mWindowManager;

	typedef std::map<sf::Key::Code, CEGUI::Key::Scan> KeyMap;
	typedef std::map<sf::Mouse::Button, CEGUI::MouseButton> MouseButtonMap;

	KeyMap mKeyMap;
	MouseButtonMap mMouseButtonMap;

	void initializeMaps();
	CEGUI::Key::Scan toCEGUIKey(sf::Key::Code Code);
	CEGUI::MouseButton toCEGUIMouseButton(sf::Mouse::Button Button);
};

#endif
```

Finally, the last class. It's also the most important, so pay attention!  
Most of this is CEGUI specific. But you may be wondering what all this is:
```cpp
typedef std::map<sf::Key::Code, CEGUI::Key::Scan> KeyMap;
typedef std::map<sf::Mouse::Button, CEGUI::MouseButton> MouseButtonMap;
 
KeyMap mKeyMap;
MouseButtonMap mMouseButtonMap;
 
void initializeMaps();
CEGUI::Key::Scan toCEGUIKey(sf::Key::Code Code);
CEGUI::MouseButton toCEGUIMouseButton(sf::Mouse::Button Button);
```
Okay, it's a *little* ugly. The problem is CEGUI and SFML use different key code constants. So we have to 'translate' between them. The same is true of mouse buttons.  
So what we do is define a std::map that maps the SFML keys to the CEGUI ones. Let's move on to all that.

### GUIManager.cpp

There is a lot here. We'll look at a few specific parts.
```cpp
bool GUIManager::Initialize(sf::RenderWindow* Win)
{
	mWindow = Win;
	mInput = &Win->GetInput();
	initializeMaps();
	try
	{
		mRenderer = new CEGUI::OpenGLRenderer(0, App::Width, App::Height);
		mSystem = new CEGUI::System(mRenderer);
		CEGUI::SchemeManager::getSingleton().create("WindowsLook.scheme");
		CEGUI::FontManager::getSingleton().createFont("DejaVuSans-10.font");

		mSystem->setDefaultMouseCursor("WindowsLook", "MouseArrow");
		mWindowManager = CEGUI::WindowManager::getSingletonPtr();
	}
	catch (CEGUI::Exception& e)
	{
		printf("CEGUI Error: %s\n", e.getMessage().c_str());
		return false;
	}
	return true;
}
```

Finally, the **Initialize** method. First, we call **initializeMaps** to fill our key/mouse button maps up. Then we just set up CEGUI. We also try to catch any CEGUI::Exceptions that might be thrown.
```cpp
bool GUIManager::handleEvent(sf::Event& Event)
{
	switch (Event.Type)
	{
	case sf::Event::TextEntered:
		return mSystem->injectChar(Event.Text.Unicode);
	case sf::Event::KeyPressed:
		return mSystem->injectKeyDown(toCEGUIKey(Event.Key.Code));
	case sf::Event::KeyReleased:
		return mSystem->injectKeyUp(toCEGUIKey(Event.Key.Code));
	case sf::Event::MouseMoved:
		return mSystem->injectMousePosition(static_cast<float>(mInput->GetMouseX()), static_cast<float>(mInput->GetMouseY()));
	case sf::Event::MouseButtonPressed:
		return mSystem->injectMouseButtonDown(toCEGUIMouseButton(Event.MouseButton.Button));
	case sf::Event::MouseButtonReleased:
		return mSystem->injectMouseButtonUp(toCEGUIMouseButton(Event.MouseButton.Button));
	case sf::Event::MouseWheelMoved:
		return mSystem->injectMouseWheelChange(static_cast<float>(Event.MouseWheel.Delta));
	}
	return false;
}
```

This is an important method. This lets CEGUI handle any events it needs to handle. If CEGUI consumes the event, this method returns true. This lets us know that we should NOT respond to the event.
```cpp
void GUIManager::Draw()
{
	mSystem->renderGUI();
}
```

Yes, that's all there is to that.

```cpp
/************************************************************************/
/*                              Private                               * /
/************************************************************************/
CEGUI::Key::Scan GUIManager::toCEGUIKey(sf::Key::Code Code)
{
	if (mKeyMap.find(Code) == mKeyMap.end())
		return (CEGUI::Key::Scan)0;

	return mKeyMap[Code];
}

CEGUI::MouseButton GUIManager::toCEGUIMouseButton(sf::Mouse::Button Button)
{
	if (mMouseButtonMap.find(Button) == mMouseButtonMap.end())
		return (CEGUI::MouseButton)0;

	return mMouseButtonMap[Button];
}

//Auto-generated (phew...), edited by hand
void GUIManager::initializeMaps()
{
	mKeyMap[sf::Key::Escape]          = CEGUI::Key::Escape       ;
	mKeyMap[sf::Key::Num1]            = CEGUI::Key::One          ;
	mKeyMap[sf::Key::Num2]            = CEGUI::Key::Two          ;
        ...

	mMouseButtonMap[sf::Mouse::Left]	= CEGUI::LeftButton;
	mMouseButtonMap[sf::Mouse::Middle]	= CEGUI::MiddleButton;
	mMouseButtonMap[sf::Mouse::Right]	= CEGUI::RightButton;
	mMouseButtonMap[sf::Mouse::XButton1]	= CEGUI::X1Button;
	mMouseButtonMap[sf::Mouse::XButton2]	= CEGUI::X2Button;
}
```

I stripped out almost all of the initializeMaps() function as it was too long. Anyways, it's clear what is going on here. These are our functions to translate keys from sf::Key::Code(s) to CEGUI::Key::Scan(s). We also translate from sf::Mouse::Button to CEGUI::MouseButton. For the full source, download the example.

## Download 
sfmlCEGUI.zip - <http://www.sendspace.com/file/0m0hws>