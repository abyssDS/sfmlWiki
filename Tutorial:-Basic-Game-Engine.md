# Creating a Basic Game Engine
<a name="top" />
At the core of every good game is a game engine. But as a beginner, building your first game engine can be pretty daunting. There are lots of things to consider when designing a solid game engine. Managing all the graphics, sounds, music, and game objects takes a lot of effort and is not as fun as designing what the game will actually do. Most people interested in game programming start with a game idea and don't know where to start in bringing their game idea into reality. And quite frankly, many of these game ideas suck. Many are just minor variations of existing games that people have already played. But this should not stop people from trying to create games, because with every half finished game comes experience and sometimes you discover that the joy of game programming comes in the process of seeing your game (good or bad) come to life at your hands. For me, that is what I seek, the *journey* of seeing my game creations come to life. For this purpose, I have decided to create a basic game engine that I can use for future game ideas, so I can more easily get to the fun part of game creation: the Game Mechanics. But before I share with you the features of my basic game engine, I want to share with you a few hard lessons I have learned over the years starting with an explanation of Namespaces.

### Quick Links ###

1. [Namespaces](#namespaces)
2. [Forward Declarations](#declarations)
3. [Main function](#mainfunc)
4. [Game Application](#gameapp)
5. [Game Loop](#gameloop)
6. [Game State](#gamestate)
7. [Manager classes](#manager)
8. [Configuration files](#config)
9. [Comments](#comments)

## <a name="namespaces" />Namespaces [ [Top] ](#top)

Most Game Engines begin with basic building blocks that are borrowed, stolen (hopefully not!), or written by other people. SFML is an example of a building block that provides basic Graphics, Sound, Network, System, and Image support. All of the classes in SFML are wrapped in a container known as a Namespace called `sf`. This is why every class, enumeration, constant, and variable is prefaced with `sf::`. For us, we will wrap our game engine in the Namespace of GQE (which stands for GatorQue Engine, after my nick GatorQue). Feel free to use a different Namespace for your project, but keep it simple and short if you do, it saves typing. To wrap a class inside of a namespace you simply do the following for the .hpp file.


```cpp
namespace MyStuff
{
  class MyClass {
    public:
      MyClass();
      virtual ~MyClass();
  };
} // namespace MyStuff
```

…and for the .cpp file you do the following:


```cpp
namespace MyStuff
{
  MyClass::MyClass()
  {
  }
 
  MyClass::~MyClass()
  {
  }
} // namespace MyStuff
```

Another great feature about Namespaces is that they prevent two identically named classes from interfering with each other. For example, in SFML there is a class called Clock. But you may decide to create your own Clock class that is completely different from the `sf::Clock` class. If you use a Namespace around your Clock class then you can help the compiler determine which Clock class you really want by prefacing the Clock class declaration with a namespace as shown below:


```cpp
class MyClass {
  public:
    MyClass();
    virtual ~MyClass();
 
    // Variables
    sf::Clock mClock1;
    MyStuff::Clock mClock2;
};
```


Notice how we can create variables of *both* Clock classes by using their Namespace tag. Because typing the namespace MyStuff in front of every variable gets tedious it is often helpful to put a single line at the top of your .cpp or .hpp file that will tell the compiler that you are using EVERY class inside of some Namespace as shown here:


```cpp
using namespace MyStuff;
// OR
using namespace sf;
```

The problem with this approach is that it will make ALL classes in that Namespace visible without their Namespace tag. So if you have a local class with the same name, the compiler will get confused. I prefer to only select the classes that I want by doing the following:


```cpp
using MyStuff::MyClass;
using sf::Clock;
```

This way, you can select specific classes you will use in the current file without bringing in the potentially conflicting classes. Now that you understand Namespaces, the next topic is how do you deal with sheer number of files you will need for a large project like a Game Project. Easy, use Forward Declarations and pointers.

## <a name="declarations" />Forward Declarations [ [Top] ](#top)
Often in Game Programming, all of your Game objects need access to their parent class in order to see other Game objects. But sometimes the parent class is the container where all of these Game objects were created. For example, you may have a Level class that contains all the Game objects in that level. But these Game objects need to keep within the boundaries of the Level class by using the boundary limit variables in the Level class. But they also need to cause other Game objects in the Level class to move, defend themselves, or basically change their current state in some way. At the code level this presents a problem as shown below:


```cpp
#include "GameObject.hpp"
class Level {
  public:
    GameObject mObjects[100];
};
```

Notice how the Level.hpp file tries to include GameObject.hpp, but see what happens when the GameObject tries to refer to its parent class of Level shown below:


```cpp
#include "Level.hpp"
class GameObject {
  public:
    Level& mParent;
};
```

When you try to compile this example the compiler will either load the Level.hpp file first or the GameObject.hpp file first. In either case, the other .hpp file will be included and create an endless cycle of one file including the other. Eventually the compiler will quit (or worse) and not give you the results you were looking for. But don't worry, the answer is simple. Forward declare the classes you will use in your .hpp files and only use include statements in your .cpp file. Here is the same example above, but using forward declarations instead:


```cpp
// Forward declare the GameObject class
class GameObject; // Notice there is no definition of what the class looks like, this comes later in the cpp file
 
class Level {
  public:
    GameObject* mObjects; // Notice how GameObjects is a pointer now, not a full object
};
```

Now, see what happens in the GameObject.hpp file


```cpp
// Forward declare the Level class
class Level; // Notice there is no include for the Level.hpp file, this will be done in the cpp file
 
class GameObject {
  public:
    Level& mParent;
}
```

Now what happens is the compiler will see that GameObject or Level are both Classes and will be looking for their definitions when they are defined. Also, since the variables themselves are pointers or address references, the compiler knows what size they need to be (32 bits or 64 bits, depending on your target CPU architecture) and happily makes them that size. Forward declarations are handy for these exact scenarios, but you must be willing to deal with Pointer or Reference objects and any necessary pointer checks throughout your code. In summary, here are some simple rules that will help you decide when to use Forward Declaration in your HPP files:

1. Is this class only used as an argument in the methods of my class? Can I change this argument to be a pointer to this class?
2. Will I know the address of the dependent class (e.g. Level) this class at Construction time of the other class (e.g. GameObject)? Then use a Reference and not a Pointer for the dependent class (e.g. Level).
3. Will these two classes need to use each other in the code? Are all of the uses of this class pointers or references?
4. Is this variable a pointer or reference address to a class?

If you answer yes to these questions, you should consider using Forward Declaration. This is also the reason why you should try to put all of your code in the CPP class and only the class declaration in the HPP file. Now that we have discussed the basics of Forward Declarations lets get to discussing the Basic Game engine I have created and the underlying features.

## <a name="mainfunc" />Main function [ [Top] ](#top)
For me, I try to keep my main.cpp files simple and straightforward. This is done by creating a single class that represents the Game Application and instantiating this class in my main function (see the [[GQE Project|http://code.google.com/p/gqe/]] for full source) as shown below:

```cpp
/**
 * This is the starting point for all new projects.  This file's purpose is
 * pretty small, but important.  In here we create our application and begin
 * the primary game loop.
 *
 * @file main.cpp
 * @author Ryan Lindeman
 * @date 20100707 - Initial Release
 * @date 20110611 - Added new logging capabilities using macros and c++ classes.
 */

#include <assert.h>
#include <stddef.h>
#include <GQE/Core.hpp>
#include <MyApplication.hpp>

int main(int argc, char* argv[])
{
  // Default anExitCode to a specific value
  int anExitCode = GQE::StatusNoError;
 
  // Create our Logger first before creating our application
  GQE::FileLogger anLogger("output.txt", true);

  // Create our action application.
  GQE::IApp* anApp = new(std::nothrow) GQE::MyApplication();
  assert(NULL != anApp && "main() Can't create Application");
 
  // Process command line arguments
  anApp->ProcessArguments(argc, argv);
 
  // Start the action application:
  // Initialize the action application
  // Enter the Game Loop where the application will remain until it is shutdown
  // Cleanup the action application
  // Exit back to here
  anExitCode = anApp->Run();
 
  // Cleanup ourselves by deleting the action application
  delete anApp;
 
  // Don't keep pointers to objects we have just deleted
  anApp = NULL;
 
  // return our exit code
  return anExitCode;
}
```

Because the main function above is so generic, you should have no trouble copying this exact file for every game you write and changed only the creation of the Application file and Include line. Lets look under the hood of the App class and see what makes it tick.

## <a name="gameapp" />Game Application [ [Top] ](#top)
In order for our basic game engine to work for any game we write, we need to determine the most generic game application algorithm to put into our Game Application class App. To do this, I took examples of other open source game engines, game engine tutorials, and personal games I have written to find the most common algorithm that works for each of them. The Game Application algorithm is outlined as follows from the App.cpp file (see the [[GQE Project|http://code.google.com/p/gqe/]] for full source):


```cpp
  int App::Run(void)
  {
    SLOG(App_Run,SeverityInfo) << std::endl;
 
    // First set our Running flag to true
    mRunning = true;
 
    // Register our App pointer with our StatManager
    mStatManager.RegisterApp(this);

    // Register our App pointer with our StateManager
    mStateManager.RegisterApp(this);
 
    // First register the IAssetHandler derived classes in the GQE Core library
    mAssetManager.RegisterHandler(new(std::nothrow) ConfigHandler());
    mAssetManager.RegisterHandler(new(std::nothrow) FontHandler());
    mAssetManager.RegisterHandler(new(std::nothrow) ImageHandler());
    mAssetManager.RegisterHandler(new(std::nothrow) MusicHandler());
    mAssetManager.RegisterHandler(new(std::nothrow) SoundHandler());
 
    // Give derived class a time to register custom IAssetHandler classes
    InitAssetHandlers();
 
    // Attempt to open the application wide settings.cfg file as a ConfigAsset
    // registered under the ID of "resources/settings.cfg"
    InitSettingsConfig();

    // Try to open the Renderer window to display graphics
    InitRenderer();
 
    // Give the derived application a chance to register a IScreenFactory class
    // to provide IScreen derived classes (previously known as IState derived
    // classes) as requested.
    InitScreenFactory();

    // Give the StatManager a chance to initialize
    mStatManager.DoInit();

    // GameLoop if Running flag is still true
    GameLoop();

    // Cleanup our application
    HandleCleanup();
 
    // Perform our own internal Cleanup
    Cleanup();

    // Make sure our Running flag is set to false before exiting
    mRunning = false;
 
    if(mExitCode < 0)
      SLOGR(App_Run,SeverityError) << "exitCode=" << mExitCode << std::endl;
    else
      SLOGR(App_Run,SeverityInfo) << "exitCode=" << mExitCode << std::endl;

     // Return the Exit Code specified by Quit or 0 of Quit was never called
    return mExitCode;
  }
```

As you can see, the algorithm consists of the following basic steps:

1. Make sure every Manager class (more about Manager classes later) has a pointer/reference to its' parent Game Application class
2. Register each AssetHandler class with the AssetManager class (more about AssetHandler classes later)
3. Open the game's configuration file and retrieve any game settings needed including those necessary for creating the Render window
4. Initialize the SFML rendering window/targets
5. Initialize the game specific information which includes creating the initial game states (more about Game states later)
6. Start running the actual game loop
7. Do any cleanup required before exiting the Game Application
8. Quit the Game Application

These steps are adequate for creating any type of game regardless of what game it might be. The only step that might need to change is the InitAssetHandlers, InitScreenFactory, and HandleCleanup steps, but all the others should be the same for every game you write (at least that is the goal). The second purpose of the Game Application class is to serve as the holding container for all our common classes that are shared throughout the entire game. Our game sf::RendererWindow class, game objects, game States, game Manager classes, etc. Before we dive into the details of the game Manager classes and game States, I want to cover the Game Loop algorithm found in the App::GameLoop method next.

## <a name="gameloop" />Game Loop [ [Top] ](#top)
Most Game Engine tutorials give you the following Game Loop algorithm:

1. Process Input devices (keyboard, mouse, joystick, etc)
2. Process Game Logic (update position, velocity, etc. of each moving game object and perform collision detection and AI functions)
3. Erase the screen and draw each game object
4. Repeat until the game end signal is set (usually set during processing of Input devices)

There is only one problem with this Game Loop algorithm: It ties your processing of Game Logic to the speed in which the computer can display graphics to the screen. If you run your game on a computer with a slow graphics card, the game logic will run at a slower speed then if you run your game on a computer with a faster graphics card. To prevent this from happening I scoured the internet for the solution to this problem (see [entropyinteractive.com](http://entropyinteractive.com/2011/02/game-engine-design-the-game-loop/)) and came up with the following Game Loop implementation (see the [[GQE Project|http://code.google.com/p/gqe/]] for full source):


```cpp
  void App::GameLoop(void)
  {
    SLOG(App_Loop, SeverityInfo) << std::endl;

    // Clock used in restricting Update loop to a fixed rate
    sf::Clock anUpdateClock;

#if (SFML_VERSION_MAJOR < 2)
    // Restart/Reset our Update clock
    anUpdateClock.Reset();

    // When do we need to update next (in seconds)?
    float anUpdateNext = anUpdateClock.GetElapsedTime();
#else
    // Clock used in calculating the time elapsed since the last frame
    sf::Clock anFrameClock;

    // Restart/Reset our Update clock
    anUpdateClock.restart();

    // When do we need to update next (in milliseconds)?
    sf::Int32 anUpdateNext = anUpdateClock.getElapsedTime().asMilliseconds();
#endif

    // Make sure we have at least one state active
    if(mStateManager.IsEmpty())
    {
      // Exit with an error since there isn't an active state
      Quit(StatusAppInitFailed);
    }

    // Loop while IsRunning returns true
#if (SFML_VERSION_MAJOR < 2)
    while(IsRunning() && mWindow.IsOpened() && !mStateManager.IsEmpty())
#else
    while(IsRunning() && mWindow.isOpen() && !mStateManager.IsEmpty())
#endif
    {
      // Get the currently active state
      IState& anState = mStateManager.GetActiveState();

      // Count the number of sequential UpdateFixed loop calls
      Uint32 anUpdates = 0;

      // Process any available input
      ProcessInput(anState);

      // Make note of the current update time
#if (SFML_VERSION_MAJOR < 2)
      float anUpdateTime = anUpdateClock.GetElapsedTime();
#else
      sf::Int32 anUpdateTime = anUpdateClock.getElapsedTime().asMilliseconds();
#endif

      // Process our UpdateFixed portion of the game loop
      while((anUpdateTime - anUpdateNext) >= mUpdateRate && anUpdates++ < mMaxUpdates)
      {
        // Let the current active state perform fixed updates next
        anState.UpdateFixed();

        // Let the StatManager perfom its updates
        mStatManager.UpdateFixed();

        // Compute the next appropriate UpdateFixed time
        anUpdateNext += mUpdateRate;
      } // while((anUpdateTime - anUpdateNext) >= mUpdateRate && anUpdates <= mMaxUpdates)

      // Let the current active state perform its variable update
#if (SFML_VERSION_MAJOR < 2)
      anState.UpdateVariable(mWindow.GetFrameTime());
#else
      // Convert to floating point value of seconds for SFML 2.0
      anState.UpdateVariable(anFrameClock.restart().asSeconds());
#endif

      // Let the current active state draw stuff
      anState.Draw();

      // Let the StatManager perform its drawing
      mStatManager.Draw();

#if (SFML_VERSION_MAJOR < 2)
      // Display Render window to the screen
      mWindow.Display();
#else
      // Display Render window to the screen
      mWindow.display();
#endif

      // Handle Cleanup of any recently removed states at this point as needed
      mStateManager.HandleCleanup(); 
    } // while(IsRunning() && !mStates.empty())
  }
```

The key to making your Game Logic run at the same speed on every computer is to realize that your Game Logic should run at a specific speed independent of your graphics card. This is done by selecting a specific Game Logic rate (for the Basic Game Engine I selected 20 Hz, which means 20 times per second the Game Logic loop will be executed) and always making sure that the Game Logic loop runs as many times (up to mMaxUpdates) as needed to meet this rate before drawing to the screen. On a modern computer (like my desktop or laptop) I compute that my Game Logic is running at 20 Hz and that my display FPS (frames per second) runs at about 60 Hz. If I run this same Basic Game Engine on an older computer, I still compute my Game Logic is running at 20 Hz but that my display FPS runs between 15 to 30 Hz. This way, the Game Logic runs at the same speed regardless of my Graphics hardware at the expense of some visual stuttering during the more animated portions of the game. The variable in the Game Loop above that makes this magic happen is the mUpdateRate and the mMaxUpdate variables which is computed as follows in the App constructor:


```cpp
 mUpdateRate(1.0f / 20) // Compute Game Logic to run at 20 Hz.  You can change this to 15, 30, or some other rate you desire.
```

So the new Game Logic algorithm is as follows:

1. Get an address reference to the current game State (which we will discuss next)
2. Perform all Input processing through the current game State
3. Perform the Game Logic processing through the current game State (see UpdateFixed call above)
4. Repeat above two steps until the Game Logic rate has been met
5. Perform the variable game rate portion (see UpdateVariable call above)
5. Allow the current game State to draw its objects to the screen
6. Repeat all of the above steps until the Exit Game Loop flag is set (see *IsRunning* || *!mState.empty()* above)

So this raises the question, what is a game State and what does it do?

## <a name="gamestate" />Game State [ [Top] ](#Top)
Many modern games, especially the casual gamer variety found on many websites, are very predictable in their basic game flow. See if you agree with the following game flow found in many games today:

1. Show a Splash screen for the company or companies that made or produced the game
2. Show a Loading Please Wait screen while the graphics for the game are loaded from the hard drive or the internet
3. Welcome the player and ask for their name/nickname (for first time players)
4. Show the Main menu and allow the gamer to choose a mode of game to play (single player, multiplayer, timed, etc)
5. Present the mode of game play to the gamer until the gamer either quits the game or ends up with Game Over
6. Repeat either the last 2 or 3 steps until the player exits the Game Application

If you agree with this general outline you can see that many games perform the following linear timeline: Splash→Load→Menu→Game→Load Level→Game→Load Level→Game→Menu→Exit As a gamer you expect to be able to return to the previous timeline if your current point in the timeline suddenly quits. This means if you reach a Game Over situation, you will be returned to the Main Menu. Each of these points in the timeline can be called a game State. A game State is just a fancy term for a specific point in your overall game flow. If you have had some advanced classes at a College for Computer Science or have read several Game Tutorials you know that I am talking about Finite State Machines or FSM's (if you have never heard of this term, please go a read a tutorial about it now, its very useful). Finite State Machines are a way of expressing the flow from one game State to another game State in a succinct graphical representation. Basically, its just a diagram of circles with lines that connect the circles. Above or below each line is written the criteria necessary to transition from one circle, or game State, to another circle. For example, our first circle would be labeled Splash and represents the Splash screen displayed to the gamer when the first program starts. A second circle could be added called the Loading which represents the Loading Please Wait screen mentioned earlier. A line between these circles might be „wait 3 seconds” meaning that after showing the Splash screen for 3 seconds transition to the Loading Please Wait screen. A third circle representing the Main Menu might also be added with a line connecting the Main Menu to the Loading Please Wait circle that has the label „wait until last sound and image is loaded” next to it. At this point our circles and lines are pretty linear. But now we have many choices to choose from and many lines will leave the Main Menu circle and go to other circles. Some of these circles will be the Options screen, the Single Player campaign, the Multi-Player campaign, or the Death Match mode or whatever else our game flow dictates. But since we desire to have a Basic Game Engine cover all of these possibilities we need some easy way to manage these transitions. We could create a custom Game Loop for each of these different game States, but why copy code when you can just abstract away the parts that change. So that is what the abstract class called IState (see the [[GQE Project|http://code.google.com/p/gqe/]] for full source) does:


```cpp
   /**
     * DoInit is responsible for initializing this State.  HandleCleanup will
     * be called if mCleanup is true so Derived classes should always call
     * IState::DoInit() first before initializing their assets.
     */
    virtual void DoInit(void)
    {
      ...
    }
 
    /**
     * ReInit is responsible for Reseting this state when the 
     * StateManager::ResetActiveState() method is called.  This way a Game
     * State can be restarted without unloading and reloading the game assets
     */
    virtual void ReInit(void) = 0;

    /**
     * DeInit is responsible for marking this state to be cleaned up
     */
    void DeInit(void)
    {
      ...
    }

    /**
     * Pause is responsible for pausing this State since the Application
     * may have lost focus or another State has become activate.
     */
    virtual void Pause(void)
    {
      ...
    }

    /**
     * Resume is responsible for resuming this State since the Application
     * may have gained focus or the previous State was removed.
     */
    virtual void Resume(void)
    {
      ...
    }

    /**
     * HandleEvents is responsible for handling input events for this
     * State when it is the active State.
     * @param[in] theEvent to process from the App class Loop method
     */
    virtual void HandleEvents(sf::Event theEvent) = 0;
 
    /**
     * UpdateFixed is responsible for handling all State fixed update needs for this
     * State when it is the active State.
     */
    virtual void UpdateFixed(void) = 0;
 
    /**
     * UpdateVariable is responsible for handling all State variable update needs for this
     * State when it is the active State.
     */
    virtual void UpdateVariable(void) = 0;
 
    /**
     * Draw is responsible for handling all Drawing needs for this State
     * when it is the Active State.
     */
    virtual void Draw(void) = 0;
 
   /**
     * HandleCleanup is responsible for performing any cleanup required before
     * this State is removed.
     */
    virtual void HandleCleanup(void)
    {
      ...
    }
```

This way our Game Application class **App** need not concern itself with what the game flow actually is, it just needs to get the current game State that is running and call the methods of the game State that was derived from the IState class shown above. That is the beauty of polymorphism (a fancy term used in object oriented programming) in action. In other words, by keeping things generic, we can save ourselves a lot of time by not reinventing the wheel every time we want to start a new game. The class that handles providing the current game State is known as a Manager class. So this leads us into our next discussion, what are Manager classes?

## <a name="manager" />Manager classes [ [Top] ](#top)
A Manager class is an object oriented technique used in our Game Application to isolate specific features. Manager classes are used to perform specific functions that all game States or game Objects need and naturally live as public member variables in the Game Application class **IApp**. For example, managing which state is being shown right now or which one will be shown next is the primary purpose of the StateManager class. Managing all the loading and unloading of game assets like images, sounds, and music for our game is also the responsibility of the AssetManager class. One of the nice features about the AssetManager class is that deals with the complexity of multiple game States sharing the same image, sound, or music assets. As each game State tells the AssetManager class about the game assets it needs, the AssetManager keeps track of the number of references for these game assets. When the game State tells the AssetManager class that it no longer needs an image, sound, or music asset the AssetManager decrements the reference count and removes the game asset from memory if it is no longer referenced. Also, the AssetManager allows you to defer the loading of all or some of the game assets until a later point in time. This is helpful for creating a special game State specifically for loading game assets. When this game State becomes the current state it activates the AssetManager loader and monitors the loading of the game assets. When the loading of the game assets completes the loader game State uses the StateManager class to remove itself from being the active state and return to the previous game State. Additional Manager classes will be added to our Basic Game Engine overtime. I am already working on a WidgetManager for providing GUI support for our Basic Game Engine.

## <a name="config" />Configuration files [ [Top] ](#top)
One of the critical aspects in creating a Basic Game Engine is having the flexibility to load configuration information from a file. The ConfigReader class provides the ability to read in .INI style files and can by a game State to load any type of information from a file. The InitSettingsConfig method found in the IApp.cpp class (see the [[GQE Project|http://code.google.com/p/gqe/]] for full source) provides a simple example of using the ConfigReader class as shown below:


```cpp
  void App::InitSettings(void)
  {
    SLOG(App_InitSettingsConfig, SeverityInfo) << std::endl;
    ConfigAsset anSettingsConfig(IApp::APP_SETTINGS);
  }
```

## <a name="comments" />Comments [ [Top] ](#top)
Great work!

One detail though. The following code doesn't work as intended:


```cpp
result = new ImageAsset(theFilename, theStyle);
assert(NULL != result && "AssetManager::AddImage() unable to allocate memory");
```

If new can't get the memory, it'll throw a bad_alloc exception. If you want it to return NULL instead, you'll have to add '(nothrow)':


```cpp
result = new(std::nothrow) ImageAsset(theFilename, theStyle);
assert(NULL != result && "AssetManager::AddImage() unable to allocate memory");
```

(See <http://www.cplusplus.com/reference/std/new/nothrow/>)

//Peter Welzien//

Thank you for the comments and encouragement.  I have made the changes you recommended and appreciate learning something new!

//Ryan Lindeman//

Ryan: thanks for the article!  I've read many similar articles over the years (and experimented with many incomplete alternatives of my own) and I must say yours stands out as appropriately clear.

Peter: It appears you pasted the same block for both examples, I've adjusted it to what I imagine you were saying.

//phobius//

The Basic Game Engine has been progressing and this tutorial needed a little more updating. Please come download the engine for yourself and give it a try.

//Ryan Lindeman//

I have a question about the game logic and renderer: Would it make sense to put these into seperate threads, so you don't have to wait on the graphics card and vice versa and make them more independent of each other? 
Are there problems with this, that I am not seeing?

// RebelMoogle //

Nice article, although I have a question regarding your Forward Declaration section. Can't you just use a preprocessor directive like #pragma once or #ifndef #define to avoid the header file loop?

//Kevin Kung//