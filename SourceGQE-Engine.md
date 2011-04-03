# Basic Game Engine
<a name="top" />
The following is the start of a basic game engine intended for beginners and experienced developers to start from.  As time permits, I will add important features to this game engine and mention their uses in the [[Basic Game Engine|TutorialGQE-Engine]] tutorial wiki.  Below is the full source code for the Game Engine.  Credit would be appreciated but not required.  Please send me an email to my nick GatorQue in the forums if you have questions or recommendations.  I would also appreciate some help translating this and the tutorial into french to reach maximum exposure for the tutorial and source code.  Thanks and please don't hesitate to suggest improvements.

Quick Links | | | | 
|-------------|-------------|-------------|-------------|-------------
[main.cpp](#1) | [StatManager.cpp](#7) | [StateManager.hpp](#13) | [ConfigAsset.cpp](#19) | [MusicAsset.cpp](#25)
[App.hpp](#2) | [IState.hpp](#8) | [StateManager.cpp](#14) | [FontAsset.hpp](#20) | [SoundAsset.hpp](#26)
[App.cpp](#3) | [SplashState.hpp](#9) | [AssetManager.hpp](#15) | [FontAsset.cpp](#21) | [SoundAsset.cpp](#27)
[ConfigReader.hpp](#4) | [SplashState.cpp](#10) | [AssetManager.cpp](#16) | [ImageAsset.hpp](#22) | [GQE.hpp](#28)
[ConfigReader.cpp](#5) | [MenuState.hpp](#11) | [TAsset.hpp](#17) | [ImageAsset.cpp](#23) | [GQE_types.hpp](#29)
[StatManager.hpp](#6) | [MenuState.cpp](#12) | [ConfigAsset.hpp](#18) | [MusicAsset.hpp](#24) | [stdint.h](#30) / [Comments](#comments)




## <a name="1" />main.cpp [ [Top] ](#top)
```cpp
/**
 * This is the starting point for all new projects.  This file's purpose is
 * pretty small, but important.  In here we create our application and begin
 * the primary game loop.
 *
 * @file main.cpp
 * @author Ryan Lindeman
 * @date 20100707 - Initial Release
 */

#include <assert.h>
#include <stddef.h>
#include "GQE/GQE.hpp"

int main(int argc, char* argv[])
{
  // Default anExitCode to a specific value
  int anExitCode = GQE::StatusNoError;

  // Create our action application.
  GQE::App* anApp = new(std::nothrow) GQE::App();
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

## <a name="2" />App.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the App class in the GQE namespace which is responsible for
 * providing the App base class implementation used by all applications.
 *
 * @file GQE/classes/App.hpp
 * @author Ryan Lindeman
 * @date 20100705 - Initial Release
 * @date 20110128 - Add Get/SetUpdateRate methods for changing game rate speed
 * @date 20110128 - Added new StatManager class for collecting game statistics
 */
#ifndef   APP_HPP_INCLUDED
#define   APP_HPP_INCLUDED

#include <fstream>
#include <stddef.h>
#include <string.h>
#include <vector>
#include "GQE/GQE_types.hpp"
#include "GQE/classes/AssetManager.hpp"
#include "GQE/classes/StatManager.hpp"
#include "GQE/classes/StateManager.hpp"
#include <SFML/Window.hpp>
#include <SFML/Graphics.hpp>

namespace GQE
{
  class App
  {
  public:
    // Constants
    ///////////////////////////////////////////////////////////////////////////
    static const unsigned int DEFAULT_VIDEO_WIDTH = 1024;
    static const unsigned int DEFAULT_VIDEO_HEIGHT = 768;
    static const unsigned int DEFAULT_VIDEO_BPP = 32;

    // Variables
    ///////////////////////////////////////////////////////////////////////////
    /// Title to use for Window
    std::string               mTitle;
    /// Video Mode to use (width, height, bpp)
    sf::VideoMode             mVideoMode;
    /// Render window to draw to
    sf::RenderWindow          mWindow;
    /// Window settings to use when creating Render window
    sf::WindowSettings        mWindowSettings;
    /// Window style to use when createing Render window
    unsigned long             mWindowStyle;
    /// Input manager for Render window above
    const sf::Input&          mInput;
    /// Output Logger file
    std::ofstream             mLog;
    /// AssetManager for managing assets
    AssetManager              mAssetManager;
    /// StatManager for managing game statistics
    StatManager               mStatManager;
    /// StateManager for managing states
    StateManager              mStateManager;

    /**
     * App constructor
     * @param[in] theTitle is the title of the window
     */
    App(const std::string theTitle = "GQE Application");

    /**
     * App deconstructor
     */
    virtual ~App();

    /**
     * ProcessArguments is responsible for processing command line arguments
     * provided to the application.
     * @param[in] argc is the number of arguments
     * @param[in] argv are the actual arguments
     */
    virtual void ProcessArguments(int argc, char* argv[]);

    /**
     * Run is called after the Application is created and will call the
     * Init, Loop, and Cleanup methods that are defined by the derived
     * class.
     */
    int Run(void);

    /**
     * IsRunning will return true if the Application is still running.
     * @return true if Application is running, false otherwise
     */
    bool IsRunning(void) const;

    /**
     * GetUpdateRate will return the current game loop update rate being
     * used.
     * @return update rate in Hz (updates per second)
     */
    float GetUpdateRate(void) const;

    /**
     * SetUpdateRate will set the game loop update rate to theRate specified
     * from 1 Hz to 1000 Hz.  Any other value outside this range will not be
     * accepted.
     * 
     * @param[in] theRate in Hz (updates per second) range is [1,1000]
     */
    void SetUpdateRate(float theRate);

    /**
     * Quit will signal the Application to stop running.
     */
    void Quit(int theExitCode);

  protected:
    /**
     * PreInit is responsible for getting things ready before the derived
     * classes Init method is called.  This prevents problems that might occur
     * with how the derived classes Init methods are written.
     */
    void PreInit(void);

    /**
     * Init is responsible for initializing the Application.
     */
    virtual void Init(void);

    /**
     * Loop is responsible for monitoring IsRunning and exiting when the
     * Application is done.
     */
    virtual void Loop(void);

    /**
     * Cleanup is responsible for performing any last minute Application
     * cleanup steps before exiting the Application.
     */
    virtual void Cleanup(void);

  private:
    /// The exit code value that will be returned by the program
    int                       mExitCode;
    /// True if the Application is currently running
    bool                      mRunning;
    /// Update rate in seconds (1.0f / UpdateRate) to use for game loop
    float                     mUpdateRate;
    /// Logger output path and filename
    std::string               mLogFile;

    /**
     * App copy constructor is private because we do not allow copies of
     * our Singleton class
     */
    App(const App&);                 // Intentionally undefined

    /**
     * Our assignment operator is private because we do not allow copies
     * of our Singleton class
     */
    App& operator=(const App&);      // Intentionally undefined
  }; // class App
}; // namespace GQE

#endif // APP_HPP_INCLUDED
```

## <a name="3" />App.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the App class in the GQE namespace which is responsible for
 * providing the App base class implementation used by all applications.
 *
 * @file GQE/classes/App.cpp
 * @author Ryan Lindeman
 * @date 20100705 - Initial Release
 * @date 20110106 - Added ConfigReader for loading window settings in PreInit
 * @date 20110118 - Fixed compiler problems with ConfigReader
 * @date 20110125 - Drop use of non-standard _getcwd for now.
 * @date 20110125 - IState::HandleCleanup is now called from StateManager
 * @date 20110128 - Add Get/SetUpdateRate methods for changing game rate speed
 * @date 20110128 - Added new StatManager class for collecting game statistics
 */
 
#include <assert.h>
#include <direct.h>
#include "GQE/classes/App.hpp"
#include "GQE/classes/ConfigReader.hpp"
#include "GQE/states/MenuState.hpp"
#include "GQE/states/SplashState.hpp"
 
namespace GQE
{
  App::App(const std::string theTitle) :
    mTitle(theTitle),
    mVideoMode(DEFAULT_VIDEO_WIDTH, DEFAULT_VIDEO_HEIGHT, DEFAULT_VIDEO_BPP),
    mWindow(),
    mWindowSettings(),
    mWindowStyle(sf::Style::Close | sf::Style::Resize),
    mInput(mWindow.GetInput()),
    mAssetManager(),
    mStatManager(),
    mStateManager(),
    mExitCode(0),
    mRunning(false),
    mUpdateRate(1.0f / 100)
  {
    mLogFile.assign("output.txt");
    mLog.open(mLogFile.c_str());
    mLog << "LogFile: " << mLogFile << std::endl;
 
    // Output to log file
    mLog << "App::App() ctor called" << std::endl;
  }
 
  App::~App()
  {
    mRunning = false;
    // Output to log file
    mLog << "App::~App() dtor called" << std::endl;
  }
 
  void App::ProcessArguments(int argc, char* argv[])
  {
    // Handle command line arguments
    // TODO: Add handling of command line arguments
    mLog << "App::ProcessArguments() Program: " << argv[0] << std::endl;
    mLog << "App::ProcessArguments() Command Line: ";
    for(int iloop = 1; iloop<argc; iloop++)
    {
      mLog << argv[iloop] << ", ";
    }
    mLog << std::endl;
  }
 
  int App::Run(void)
  {
    // Log the starting of Run
    mLog << "App::Run() starting" << std::endl;
 
    // First set our Running flag to true
    mRunning = true;
 
    // Register our App pointer with our AssetManager
    mAssetManager.RegisterApp(this);
 
    // Register our App pointer with our StatManager
    mStatManager.RegisterApp(this);

    // Register our App pointer with our StateManager
    mStateManager.RegisterApp(this);
 
    // Pre-init is responsible for the following:
    // 1) Opening our configuration file
    // 2) Setting up our render window
    PreInit();
 
    // Initialize our application which might set our Running flag to false
    Init();
 
    // Loop if Running flag is still true
    Loop();
 
    // Cleanup our application
    Cleanup();
 
    // Make sure our Running flag is set to false before exiting
    mRunning = false;
 
    // Log our Exit Code value
    mLog << "App::Run() returning with " << mExitCode << std::endl;
 
    // Return the Exit Code specified by Quit or 0 of Quit was never called
    return mExitCode;
  }
 
  bool App::IsRunning(void) const
  {
    return mRunning;
  }
 
  float App::GetUpdateRate(void) const
  {
    return (1.0f / mUpdateRate);
  }

  void App::SetUpdateRate(float theRate)
  {
    if(1000.0f >= theRate && 1.0f <= theRate)
    {
      mUpdateRate = 1.0f / theRate;
    }
  }

  void App::Quit(int theExitCode)
  {
    mExitCode = theExitCode;
    mRunning = false;
  }
 
  void App::PreInit(void)
  {
    ConfigReader anConfig;       // For reading .INI style files
 
    // Use our default configuration file to obtain the initial window settings
    anConfig.RegisterApp(this);  // For logging purposes, let ConfigReader know about us
    anConfig.Read("window.cfg"); // Read in our window settings
 
    // Are we in Fullscreen mode?
    if(anConfig.GetBool("window","fullscreen",false))
    {
      mWindowStyle = sf::Style::Fullscreen;
    }
 
    // What size window does the user want?
    mVideoMode.Width = anConfig.GetUint32("window","width",DEFAULT_VIDEO_WIDTH);
    mVideoMode.Height = anConfig.GetUint32("window","height",DEFAULT_VIDEO_HEIGHT);
    mVideoMode.BitsPerPixel = anConfig.GetUint32("window","depth",DEFAULT_VIDEO_BPP);
 
    // For Fullscreen, verify valid VideoMode, otherwise revert to defaults for Fullscreen
    if(sf::Style::Fullscreen == mWindowStyle && false == mVideoMode.IsValid())
    {
      mVideoMode.Width = DEFAULT_VIDEO_WIDTH;
      mVideoMode.Height = DEFAULT_VIDEO_HEIGHT;
      mVideoMode.BitsPerPixel = DEFAULT_VIDEO_BPP;
    }
 
    // Create a RenderWindow object using VideoMode object above
    mWindow.Create(mVideoMode, mTitle, mWindowStyle, mWindowSettings);
 
    // Use Vertical Sync
    mWindow.UseVerticalSync(true);
 
    // Output to log file
    mLog << "App::PreInit() completed" << std::endl;
  }
 
  void App::Init(void)
  {
    // Give the StatManager a chance to initialize
    mStatManager.DoInit();

    // Show statistics: Frames per second (FPS) and Updates per second (UPS)
    mStatManager.SetShow(true);

    // Add Menu State as the next active state
    mStateManager.AddActiveState(new(std::nothrow) MenuState(this));
 
    // Add Splash State as current active state
    mStateManager.AddActiveState(new(std::nothrow) SplashState(this));
 
    // Output to log file
    mLog << "App::Init() completed" << std::endl;
  }
 
  void App::Loop(void)
  {
    // Clock used in restricting Update loop to a fixed rate
    sf::Clock anUpdateClock;
    anUpdateClock.Reset();
 
    // When do we need to update next?
    float anUpdateNext = anUpdateClock.GetElapsedTime();
 
    // Make sure we have at least one state active
    if(mStateManager.IsEmpty())
    {
      // Exit with an error since there isn't an active state
      Quit(StatusAppInitFailed);
    }
 
    // Loop while IsRunning returns true
    while(IsRunning() && mWindow.IsOpened() && !mStateManager.IsEmpty())
    {
      // Get the currently active state
      IState* anState = mStateManager.GetActiveState();
 
      // Check for corrupt state returned by our StateManager
      assert(NULL != anState && "App::Loop() received a bad pointer");
 
      // Create a fixed rate Update loop
      while(anUpdateClock.GetElapsedTime() > anUpdateNext)
      {
        // Handle some events and let the current active state handle the rest
        sf::Event anEvent;
        while(mWindow.GetEvent(anEvent))
        {
          // Switch on Event Type
          switch(anEvent.Type)
          {
          case sf::Event::Closed:       // Window closed
            Quit(StatusAppOK);
            break;
          case sf::Event::GainedFocus:  // Window gained focus
            anState->Resume();
            break;
          case sf::Event::LostFocus:    // Window lost focus
            anState->Pause();
            break;
          case sf::Event::Resized:      // Window resized
            break;
          default:                      // Current active state will handle
            anState->HandleEvents(anEvent);
          } // switch(anEvent.Type)
        } // while(mWindow.GetEvent(anEvent))
 
        // Let the current active state perform updates next
        anState->Update();
 
        // Let the StatManager perfom its updates
        mStatManager.Update();

        // Update our update next time
        anUpdateNext += mUpdateRate;
      } // while(anUpdateClock.GetElapsedTime() > anUpdateNext)
 
      // Let the current active state draw stuff
      anState->Draw();
 
      // Let the StatManager perform its drawing
      mStatManager.Draw();

      // Display Render window to the screen
      mWindow.Display();

      // Handle Cleanup of any recently removed states at this point as needed
      mStateManager.HandleCleanup(); 
    } // while(IsRunning() && !mStates.empty())
  }
 
  void App::Cleanup(void)
  {
    // Give the StatManager a chance to de-initialize
    mStatManager.DeInit();

    // Close the Render window if it is still open
    if(mWindow.IsOpened())
    {
      // Show the Mouse cursor
      mWindow.ShowMouseCursor(true);
 
      // Close the Render window
      mWindow.Close();
    }
 
    // Output to log file
    mLog << "App::Cleanup() completed" << std::endl;
  }
 
}; // namespace GQE
```

## <a name="4" />ConfigReader.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the ConfigReader class in the GQE namespace which is responsible
 * for providing the reading of configuration files.
 *
 * @file GQE/classes/ConfigReader.hpp
 * @author Ryan Lindeman
 * @date 20110101 - Initial Release
 * @date 20110118 - Fixed compiler problems.
 * @date 20110131 - Added class and method argument documentation
 * @date 20110218 - Added boolean result to Read method for success
 */
#ifndef   CONFIG_READER_HPP_INCLUDED
#define   CONFIG_READER_HPP_INCLUDED
 
#include <map>
#include <vector>
#include <string>
#include <stdint.h>
#include "GQE/GQE_types.hpp"
#include "SFML/Graphics/Color.hpp"
 
namespace GQE
{
  class ConfigReader
  {
  public:
    /**
     * ConfigReader constructor
     */
    ConfigReader();
 
    /**
     * ConfigReader deconstructor
     */
    virtual ~ConfigReader();
 
    /**
     * RegisterApp will register a pointer to the App class so it can be used
     * by the WidgetManager for error handling and log reporting.
     * @param[in] theApp is a pointer to the App (or App derived) class
     */
    void RegisterApp(App* theApp);
 
    /**
     * IsSectionEmpty determines if theSection provided exists and has 1 or
     * more name, value pairs to retrieve.
     * @param[in] theSection to check
     * @return true if theSection provided exists
     */
    bool IsSectionEmpty(const std::string theSection);
 
 
    /**
     * GetBool will return the boolean value for theSection and theName
     * specified or theDefault(false) if the section or name does not
     * exist.  The value must be one of the following to return either
     * true or false (0,1,on,off,true,false).
     * @param[in] theSection to use for finding the value to return
     * @param[in] theName to use for finding the value to return
     * @param[in] theDefault to use if the value is not found (optional)
     * @return the value found or theDefault if not found or correct
     */
    bool GetBool(const std::string theSection, const std::string theName,
      const bool theDefault = false);
 
    /**
     * GetString will return the string value for theSection and theName
     * specified or theDefault("") if the section or name does not exist.
     * @param[in] theSection to use for finding theName
     * @param[in] theName to use for finding the value to return
     * @param[in] theDefault to use if the value is not found (optional)
     * @return the value found or theDefault if not found
     */
    std::string GetString(const std::string theSection,
      const std::string theName, const std::string theDefault = "");
 
    /**
     * GetUint32 will return an unsigned 32 bit number for theSection and
     * theName specified or theDefault(0) if theSection or theName does not
     * exist.
     * @param[in] theSection to use for finding theName
     * @param[in] theName to use for finding the value to return
     * @param[in] theDefault to use if the value is not found (optional)
     * @return the value found or theDefault if not found
     */
    uint32_t GetUint32(const std::string theSection, const std::string theName,
      const uint32_t theDefault = 0);
 
    /**
     * GetColor will return the color value for the section and name
     * specified or theDefault(0,0,0,0) if the section or name does not exist.
     * @param[in] theSection to use for finding theName
     * @param[in] theName to use for finding the value to return
     * @param[in] theDefault color to use if the value is not found or is
     *   indecipherable (optional)
     * @return the color found or theDefault if not found
     */
    sf::Color GetColor(const std::string theSection, const std::string theName,
      const sf::Color theDefault = sf::Color::Black);
 
    /**
     * Read will open and read the configuration file specified into internal
     * maps that can be later retrieved using the Get* options below.
     * @param[in] theFilename to use as the configuration file to read
     * @result true if theFilename was found and opened successfully
     */
    bool Read(const std::string theFilename);
 
  private:
    // Constants
    ///////////////////////////////////////////////////////////////////////////
    static const unsigned short MAX_CHARS = 100;
 
    // Variables
    ///////////////////////////////////////////////////////////////////////////
    /// Pointer to the App class for error handling and logging
    App*                  mApp;
    /// Map to store all the sections and their corresponding name=value pairs
    std::map<const std::string, typeNameValue*> mSections;
 
    /**
     * ParseBool will parse theValue string to obtain the boolean value to
     * return.  If the value is not one of the following (0,1,true,false,on,
     * off) then theDefault will be returned instead.
     * @param[in] theValue to parse for (0,1,true,false,on,off)
     * @param[in] theDefault value to return if not one of the above
     * @return the boolean value obtained
     */
    bool ParseBool(const std::string theValue, const bool theDefault);
 
    /**
     * ParseColor will parse theValue string to obtain the R,G,B,A color values
     * to produce an sf::Color object for the GetColor method above.
     * @param[in] theValue to parse for R,G,B,A color values
     * @param[in] theDefault color to use if the parser fails
     * @return the color object created with the values obtained
     */
    sf::Color ParseColor(const std::string theValue,
      const sf::Color theDefault);
 
    /**
     * ParseUint32 will parse theValue string to obtain an unsigned 32 bit
     * value.  If the parser fails, then it will return theDefault instead.
     * @param[in] theValue to parse for the unsigned 32 bit value
     * @param[in] theDefault unsigned 32 bit value to use if the parser fails
     * @return the unsigned 32 bit value obtained
     */
    uint32_t ParseUint32(const std::string theValue,
      const uint32_t theDefault);
 
    /**
     * ParseLine will parse the line provided by the Read method above.
     * @param theLine to be parsed.
     * @param theCount is the line number currently being processed
     * @param theSection to be used if name, value pair is parsed
     * @return string representing the new section name if section name was parsed
     */
    std::string ParseLine(const char* theLine, const unsigned long theCount,
      const std::string theSection);
 
    /**
     * StoreNameValue will store theName and theValue pair into theSection.
     * If theSection name value map doesn't yet exist, then it will be created.
     * @param theSection is used to find the name, value pair map
     * @param theName to be stored as the key for theValue below
     * @param theValue to be stored in the current section name
     */
    void StoreNameValue(const std::string theSection,
      const std::string theName, const std::string theValue);
 
    /**
     * Our copy constructor is private because we do not allow copies
     * of our class
     */
    ConfigReader(const ConfigReader&); // Intentionally undefined
 
    /**
     * Our assignment operator is private because we do not allow copies
     * of our class
     */
    ConfigReader& operator=(const ConfigReader&); // Intentionally undefined
 
  }; // class ConfigReader
}; // namespace GQE
 
#endif // CONFIG_READER_HPP_INCLUDED
```

## <a name="5" />ConfigReader.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the ConfigReader class in the GQE namespace which is responsible
 * for providing the reading of configuration files.
 *
 * @file GQE/classes/ConfigReader.cpp
 * @author Ryan Lindeman
 * @date 20110101 - Initial Release
 * @date 20110125 - Fix string compare problems
 * @date 20110128 - Fixed erase call in the DeleteXYZ methods.
 * @date 20110218 - Added boolean result to Read method for success
 */

#include <assert.h>
#include <stddef.h>
#include <stdlib.h>
#include <sstream>
#include <string.h>
#include "GQE/classes/ConfigReader.hpp"
#include "GQE/classes/App.hpp"

namespace GQE
{
  ConfigReader::ConfigReader() :
    mApp(NULL)
  {
  }
 
  ConfigReader::~ConfigReader()
  {
    // Output to log file
    if(NULL != mApp)
    {
      mApp->mLog << "ConfigReader::~ConfigReader() dtor called" << std::endl;
    }
 
    // Delete all section name, value maps
    std::map<const std::string, typeNameValue*>::iterator iter;
    iter = mSections.begin();
    while(iter != mSections.end())
    {
      typeNameValue* anMap = iter->second;
      mSections.erase(iter++);
      delete anMap;
    }
 
    // Clear pointers we don't need anymore
    mApp = NULL;
  }
 
  void ConfigReader::RegisterApp(App* theApp)
  {
    // Check that our pointer is good
    assert(NULL != theApp && "ConfigReader::RegisterApp() theApp pointer provided is bad");
 
    // Make a note of the pointer
    assert(NULL == mApp && "ConfigReader::RegisterApp() theApp pointer was already registered");
    mApp = theApp;
  }
 
  bool ConfigReader::IsSectionEmpty(const std::string theSection)
  {
    bool anResult = false;
 
    // Check if theSection really exists
    std::map<const std::string, typeNameValue*>::iterator iter;
    iter = mSections.find(theSection);
    if(iter != mSections.end())
    {
      typeNameValue* anMap = iter->second;
      if(NULL != anMap)
      {
        anResult = anMap->empty();
      }
    }
 
    // Return the result found above or the default value of false
    return anResult;
  }
 
  bool ConfigReader::GetBool(const std::string theSection,
    const std::string theName, const bool theDefault)
  {
    bool anResult = theDefault;
 
    // Check if theSection really exists
    std::map<const std::string, typeNameValue*>::iterator iter;
    iter = mSections.find(theSection);
    if(iter != mSections.end())
    {
      // Try to obtain the name, value pair
      typeNameValue* anMap = iter->second;
      if(NULL != anMap)
      {
        typeNameValueIter iterNameValue;
        iterNameValue = anMap->find(theName);
        if(iterNameValue != anMap->end())
        {
          anResult = ParseBool(iterNameValue->second, theDefault);
        }
      }
    }
 
    // Return the result found or theDefault assigned above
    return anResult;
  }
 
  std::string ConfigReader::GetString(const std::string theSection,
    const std::string theName, const std::string theDefault)
  {
    std::string anResult = theDefault;
 
    // Check if theSection really exists
    std::map<const std::string, typeNameValue*>::iterator iter;
    iter = mSections.find(theSection);
    if(iter != mSections.end())
    {
      // Try to obtain the name, value pair
      typeNameValue* anMap = iter->second;
      if(NULL != anMap)
      {
        typeNameValueIter iterNameValue;
        iterNameValue = anMap->find(theName);
        if(iterNameValue != anMap->end())
        {
          anResult = iterNameValue->second;
        }
      }
    }
 
    // Return the result found or theDefault assigned above
    return anResult;
  }

  uint32_t ConfigReader::GetUint32(const std::string theSection,
    const std::string theName, const uint32_t theDefault)
  {
    uint32_t anResult = theDefault;

    // Check if theSection really exists
    std::map<const std::string, typeNameValue*>::iterator iter;
    iter = mSections.find(theSection);
    if(iter != mSections.end())
    {
      // Try to obtain the name, value pair
      typeNameValue* anMap = iter->second;
      if(NULL != anMap)
      {
        typeNameValueIter iterNameValue;
        iterNameValue = anMap->find(theName);
        if(iterNameValue != anMap->end())
        {
          anResult = ParseUint32(iterNameValue->second, theDefault);
        }
      }
    }

    // Return the result found or theDefault assigned above
    return anResult;
  }

  sf::Color ConfigReader::GetColor(const std::string theSection,
    const std::string theName, const sf::Color theDefault)
  {
    sf::Color anResult = theDefault;
 
    // Check if theSection really exists
    std::map<const std::string, typeNameValue*>::iterator iter;
    iter = mSections.find(theSection);
    if(iter != mSections.end())
    {
      // Try to obtain the name, value pair
      typeNameValue* anMap = iter->second;
      if(NULL != anMap)
      {
        typeNameValueIter iterNameValue;
        iterNameValue = anMap->find(theName);
        if(iterNameValue != anMap->end())
        {
          anResult = ParseColor(iterNameValue->second, theDefault);
        }
      }
    }
 
    // Return the result found or theDefault assigned above
    return anResult;
  }
 
  bool ConfigReader::Read(const std::string theFilename)
  {
    bool anResult = false;
    char anLine[MAX_CHARS];
    std::string anSection;
    unsigned long anCount = 1;
 
    // Let the log know about the file we are about to read in
    if(NULL != mApp)
    {
      mApp->mLog << "ConfigReader::Read() opening file " << theFilename << std::endl;
    }
 
    // Attempt to open the file
    FILE* anFile = fopen(theFilename.c_str(), "r");
 
    // Read from the file if successful
    if(NULL != anFile)
    {
      // Keep reading from configuration file until we reach the end of file marker
      while(!feof(anFile))
      {
        // Get the first line from the file
        if(fgets(anLine, MAX_CHARS, anFile) == NULL)
        {
          // Log the failure to read a line from the file if not at the end of the file
          if(NULL != mApp && !feof(anFile))
          {
            mApp->mLog << "ConfigReader::Read() error reading line " << anCount << " from file " << theFilename << std::endl;
          }
          // Exit our while loop, were done!
          break;
        }
        else
        {
          // Parse the line
          anSection = ParseLine(anLine, anCount, anSection);
        }
 
        // Increment our Line counter
        anCount++;
      }
 
      // Don't forget to close the file
      fclose(anFile);
      
      // Set success result
      anResult = true;
    }
    else
    {
      // Log the failure to open the file
      if(NULL != mApp)
      {
        mApp->mLog << "ConfigReader::Read() error opening file " << theFilename << std::endl;
      }
    }
    
    // Return anResult of true if successful, false otherwise
    return anResult;
  }
 
  bool ConfigReader::ParseBool(const std::string theValue,
    const bool theDefault)
  {
    bool anResult = theDefault;
    // Look for true results
    if(GQE_STRICMP(theValue.c_str(),"true") == 0 ||
       GQE_STRICMP(theValue.c_str(),"1") == 0 ||
       GQE_STRICMP(theValue.c_str(),"on") == 0)
       anResult = true;
    // Look for false results
    if(GQE_STRICMP(theValue.c_str(),"false") == 0 ||
       GQE_STRICMP(theValue.c_str(),"0") == 0 ||
       GQE_STRICMP(theValue.c_str(),"off") == 0)
       anResult = false;
 
    // Return the result found or theDefault assigned above
    return anResult;
  }
 
  sf::Color ConfigReader::ParseColor(const std::string theValue,
    const sf::Color theDefault)
  {
    sf::Color anResult = theDefault;
 
    // Try to find the first value: Red
    size_t anRedOffset = theValue.find_first_of(',');
    if(anRedOffset != std::string::npos)
    {
      sf::Uint8 anRed = (sf::Uint8)atoi(theValue.substr(0,anRedOffset).c_str());
      // Try to find the next value: Green
      size_t anGreenOffset = theValue.find_first_of(',',anRedOffset+1);
      if(anGreenOffset != std::string::npos)
      {
        sf::Uint8 anGreen = (sf::Uint8)atoi(theValue.substr(anRedOffset+1,anGreenOffset).c_str());
        // Try to find the next value: Blue
        size_t anBlueOffset = theValue.find_first_of(',',anGreenOffset+1);
        if(anBlueOffset != std::string::npos)
        {
          sf::Uint8 anBlue = (sf::Uint8)atoi(theValue.substr(anGreenOffset+1,anBlueOffset).c_str());
          sf::Uint8 anAlpha = (sf::Uint8)atoi(theValue.substr(anBlueOffset+1).c_str());
          // Now that all 4 values have been parsed, return the color found
          anResult.r = anRed;
          anResult.g = anGreen;
          anResult.b = anBlue;
          anResult.a = anAlpha;
        }
      }
    }
 
    // Return the result found or theDefault assigned above
    return anResult;
  }

  uint32_t ConfigReader::ParseUint32(const std::string theValue,
      const uint32_t theDefault)
  {
    uint32_t anResult = theDefault;
    std::istringstream iss(theValue);

    // Convert the string to an unsigned 32 bit integer
    iss >> anResult;

    // Return the result found or theDefault assigned above
    return anResult;
  }

  std::string ConfigReader::ParseLine(const char* theLine,
    const unsigned long theCount, const std::string theSection)
  {
    std::string anResult = theSection;
    size_t anLength = strlen(theLine);
    if(anLength > 0)
    {
      // Skip preceeding spaces at the begining of the line
      size_t anOffset = 0;
      while(anOffset < anLength && theLine[anOffset] == ' ')
      {
        anOffset++;
      }
 
      // Now check for comments
      if(theLine[anOffset] != '#' && theLine[anOffset] != ';')
      {
        // Next check for the start of a new section
        if(theLine[anOffset] == '[')
        {
          // Skip over the begin section marker '['
          anOffset++;
 
          // Skip preceeding spaces of section name
          while(anOffset < anLength && theLine[anOffset] == ' ')
          {
            anOffset++;
          }
 
          // Retrieve the section name while looking for the section end marker ']'
          size_t anIndex = 0;
          char anSection[MAX_CHARS] = {0};
          while((anOffset+anIndex) < anLength && theLine[anOffset+anIndex] != ']')
          {
            anSection[anIndex] = theLine[anOffset+anIndex++];
          }
          // Add null terminator
          anSection[anIndex] = '\0';
          // Remove trailing spaces
          while(anIndex > 0 && anSection[anIndex-1] == ' ')
          {
            // Put a null terminator at the end of the section name
            anSection[--anIndex] = '\0';
          }
 
          // Only update the current section name if we found the section end
          // marker before the end of the line
          if((anOffset+anIndex) < anLength && anIndex > 0)
          {
            // Change the return string to the newly parsed section name
            anResult = anSection;
          }
          else
          {
            // Log the failure to open the file
            if(NULL != mApp)
            {
              mApp->mLog << "ConfigReader::ParseLine() missing section end marker ']' on line "
                << theCount << std::endl;
            }
          }
        }
        // Just read the name=value pair into the current section
        else
        {
          // Retrieve the name and value while looking for the comment flags ';' or '#'
          size_t anNameIndex = 0;
          char anName[MAX_CHARS];
 
          // First retrieve the name while looking for either the '=' or ':' delimiter
          while((anOffset+anNameIndex) < anLength &&
            theLine[(anOffset+anNameIndex)] != '=' &&
            theLine[(anOffset+anNameIndex)] != ':')
          {
            // Grab the name
            anName[anNameIndex] = theLine[anOffset+anNameIndex++];
          }
          // Assign our starting offset value
          anOffset += anNameIndex;
          // Put a null terminator at the end of the name
          anName[anNameIndex] = '\0';
          // Remove trailing spaces from the name
          while(anNameIndex > 0 && anName[anNameIndex-1] == ' ')
          {
            // Put a null terminator at the end of the name
            anName[--anNameIndex] = '\0';
          }
 
          // Only search for the value if we found the '=' or ':' delimiter
          if(anOffset < anLength && anNameIndex > 0)
          {
            size_t anValueIndex = 0;
            char anValue[MAX_CHARS];
 
            // Skip over the delimiter between name and value '=' or ':'
            anOffset++;
 
            // Skip preceeding spaces
            while(anOffset < anLength && theLine[anOffset] == ' ')
            {
              anOffset++;
            }
            // Next retrieve the value while looking for comments flags ';' or '#'
            while((anOffset + anValueIndex) < anLength &&
              theLine[(anOffset+anValueIndex)] != '\r' &&
              theLine[(anOffset+anValueIndex)] != '\n' &&
              theLine[(anOffset+anValueIndex)] != ';' &&
              theLine[(anOffset+anValueIndex)] != '#')
            {
              // Grab the value
              anValue[anValueIndex] = theLine[anOffset+anValueIndex++];
            }
            // Put a null terminator at the end of the section name
            anValue[anValueIndex] = '\0';
            // Remove trailing spaces from the name
            while(anValueIndex > 0 && anValue[anValueIndex-1] == ' ')
            {
              // Put a null terminator at the end of the name
              anValue[--anValueIndex] = '\0';
            }
 
            // Store the name,value pair obtained into the current section
            StoreNameValue(theSection,anName,anValue);
          }
          else
          {
            // Log the failure to open the file
            if(NULL != mApp)
            {
              mApp->mLog << "ConfigReader::ParseLine() missing name or value delimiter of '=' or ':'"
                << " on line " << theCount << std::endl;
            }
          }
        }
      } // if(theLine[anOffset] != '#' && theLine[anOffset] != ';') // Not a comment
    } // if(anLength > 0)
 
    // Return either the previous section name or the new section name found
    return anResult;
  }
 
  void ConfigReader::StoreNameValue(const std::string theSection,
    const std::string theName, const std::string theValue)
  {
    // Check if the name, value map already exists for theSection
    std::map<const std::string, typeNameValue*>::iterator iterSection;
    iterSection = mSections.find(theSection);
    if(iterSection == mSections.end())
    {
      // First try to create a new name, value pair map for this new section
      typeNameValue* anMap = new (std::nothrow) typeNameValue;
 
      // Make sure we were able to create the map ok
      if(NULL != anMap)
      {
        // Log the failure to open the file
        if(NULL != mApp)
        {
          mApp->mLog << "ConfigReader::StoreNameValue() adding new section (" << theSection
            << ") the name, value pair (" << theName << "," << theValue << ")" << std::endl;
        }
 
        // Add the new name, value pair to this map
        anMap->insert(std::pair<const std::string, const std::string>(theName,theValue));
 
        // Add the new name, value pair map for this new section
        mSections.insert(std::pair<const std::string, typeNameValue*>(theSection, anMap));
      }
      else
      {
        // Log the failure to open the file
        if(NULL != mApp)
        {
          mApp->mLog << "ConfigReader::StoreNameValue() unable to create name, value map" << std::endl;
        }
      }
    }
    else
    {
      // Retrieve the existing name, value pair map
      typeNameValue* anMap = iterSection->second;
 
      // Make sure we were able to retrieve the map ok
      if(NULL != anMap)
      {
        // Make sure the name, value pair doesn't already in the map
        typeNameValueIter iterNameValue;
        iterNameValue = anMap->find(theName);
        if(iterNameValue == anMap->end())
        {
          // Log the failure to open the file
          if(NULL != mApp)
          {
            mApp->mLog << "ConfigReader::StoreNameValue() adding to existing section (" << theSection
              << ") the name, value pair (" << theName << "," << theValue << ")" << std::endl;
          }
 
          // Add the new name, value pair to this map
          anMap->insert(std::pair<const std::string, const std::string>(theName,theValue));
        }
        else
        {
          // Log the failure to open the file
          if(NULL != mApp)
          {
            mApp->mLog << "ConfigReader::StoreNameValue() name, value pair (" << theName
              << "," << theValue << ") already exists" << std::endl;
          }
        }
      }
    } // else(iterSection == mSections.end())
  }

}; // namespace GQE
```

## <a name="6" />StatManager.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the StatManager class in the GQE namespace which is responsible
 * for collecting and providing Statistical information about the application.
 * This information includes, the current Updates per second and Frames per
 * second and other statistics.
 *
 * @file GQE/classes/StatManager.hpp
 * @author Ryan Lindeman
 * @date 20110128 - Initial Release
 */
#ifndef   STAT_MANAGER_HPP_INCLUDED
#define   STAT_MANAGER_HPP_INCLUDED

#include <stdint.h>
#include "GQE/GQE_types.hpp"
#include <SFML/Graphics.hpp>
#include <SFML/System.hpp>
 
namespace GQE
{
  class StatManager
  {
  public:
 
    /**
     * StatManager constructor
     */
    StatManager();
 
    /**
     * StatManager deconstructor
     */
    virtual ~StatManager();
 
    /**
     * DoInit will reset all the statistics, load necessary fonts and
     * performing other useful things.
     */
    void DoInit(void);

    /**
     * DeInit will unload the fonts used and dump a summary of statistics
     * collected to the log file.
     */
    void DeInit(void);

    /**
     * IsShowing will return true if the current statistics are being displayed.
     * @return true if stats are being displayed, false otherwise
     */
    bool IsShowing(void) const;

    /**
     * SetShowing will either enable or disable the showing of the current
     * statistics being displayed.
     * @param[in] theShow is the new show value
     */
    void SetShow(bool theShow);

    /**
     * GetUpdate will return the current update number which is the number of
     * updates that have been called since the application started.
     * @return the number of updates since the application started.
     */
    uint32_t GetUpdates(void) const;

    /**
     * GetFrame will return the current frame number which is the number of
     * frames that have been displayed since the application started.
     * @return the number of frames since the application started.
     */
    uint32_t GetFrames(void) const;

    /**
     * RegisterApp will register a pointer to the App class so it can be used
     * by the StatManager for error handling and log reporting.
     * @param[in] theApp is a pointer to the App (or App derived) class
     */
    void RegisterApp(App* theApp);

    /**
     * Update is responsible for updating game loop statistics like Updates
     * per second.
     */
    void Update(void);
 
    /**
     * Draw is responsible for updating game loop statistics like Frames
     * per second and handling all Drawing needs for the StatManager.
     */
    void Draw(void);

  private:
    // Constants
    ///////////////////////////////////////////////////////////////////////////
 
    // Variables
    ///////////////////////////////////////////////////////////////////////////
    /// Pointer to the App class for error handling and logging
    App*        mApp;
    /// Allow the current statistics to be displayed?
    bool        mShow;
    /// Total number of frames drawn since DoInit was called
    uint32_t    mFrames;
    /// Frame clock for displaying Frames per second value
    sf::Clock   mFrameClock;
    /// Debug string to display that shows the frames per second
    sf::String  mFPS;
    /// Total number of updates done since DoInit was called
    uint32_t    mUpdates;
    /// Update clock for displaying Updates per second value
    sf::Clock   mUpdateClock;
    /// Debug string to display that shows the updates per second
    sf::String  mUPS;
 
    /**
     * StatManager copy constructor is private because we do not allow copies
     * of our class
     */
    StatManager(const StatManager&); // Intentionally undefined
 
    /**
     * Our assignment operator is private because we do not allow copies
     * of our class
     */
    StatManager& operator=(const StatManager&); // Intentionally undefined
 
  }; // class StatManager
}; // namespace GQE
 
#endif // STAT_MANAGER_HPP_INCLUDED
```

## <a name="7" />StatManager.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the StatManager class in the GQE namespace which is responsible
 * for collecting and providing Statistical information about the application.
 * This information includes, the current Updates per second and Frames per
 * second and other statistics.
 *
 * @file GQE/classes/StatManager.hpp
 * @author Ryan Lindeman
 * @date 20110128 - Initial Release
 */
 
#include <assert.h>
#include <sstream>
#include "GQE/classes/StatManager.hpp"
#include "GQE/classes/App.hpp"
 
namespace GQE
{
  StatManager::StatManager() :
    mApp(NULL),
    mShow(false),
    mFrames(0),
    mUpdates(0),
    mFPS(),
    mUPS()
  {
  }
 
  StatManager::~StatManager()
  {
    // Output to log file
    if(NULL != mApp)
    {
      mApp->mLog << "StatManager::~StatManager() dtor called" << std::endl;
    }
 
    // Clear pointers we don't need anymore
    mApp = NULL;
  }

  void StatManager::DoInit(void)
  {
    // Reset our counters
    mFrames = 0;
    mUpdates = 0;

    // Reset our clocks
    mFrameClock.Reset();
    mUpdateClock.Reset();

    // Position and color for the FPS/UPS string
    mFPS.SetColor(sf::Color(255,255,255,128));
    mFPS.Move(0,0);
    mUPS.SetColor(sf::Color(255,255,255,128));
    mUPS.Move(0,30);

    // Default strings to display for Frames/Updates per second
    mFPS.SetText("");
    mUPS.SetText("");
  }

  void StatManager::DeInit(void)
  {
  }

  bool StatManager::IsShowing(void) const
  {
    return mShow;
  }

  void StatManager::SetShow(bool theShow)
  {
    mShow = theShow;
  }

  uint32_t StatManager::GetUpdates(void) const
  {
    return mUpdates;
  }

  uint32_t StatManager::GetFrames(void) const
  {
    return mFrames;
  }

  void StatManager::RegisterApp(App* theApp)
  {
    // Check that our pointer is good
    assert(NULL != theApp && "StatManager::RegisterApp() theApp pointer provided is bad");
 
    // Make a note of the pointer
    assert(NULL == mApp && "StatManager::RegisterApp() theApp pointer was already registered");
    mApp = theApp;
  }

  void StatManager::Update(void)
  {
    // Check our App pointer
    assert(NULL != mApp && "StatManager::Update() bad app pointer");

    // Increment our update counter
    mUpdates++;
    if(mUpdateClock.GetElapsedTime() > 1.0f)
    {
      // Updates string stream
      std::ostringstream updates;

      // Update our UPS string to be displayed
      updates.precision(2);
      updates.width(7);
      updates << "UPS: " << std::fixed << (float)mUpdates / mUpdateClock.GetElapsedTime();
      mUPS.SetText(updates.str());

      // Reset our Update clock and update counter
      mUpdates = 0;
      mUpdateClock.Reset();
    }
  }

  void StatManager::Draw(void)
  {
    // Check our mApp pointer
    assert(NULL != mApp && "StatManager::Draw() invalid app pointer provided");

    // Increment our frame counter
    mFrames++;
    if(mFrameClock.GetElapsedTime() > 1.0f)
    {
      // Frames string stream
      std::ostringstream frames;

      // Get our FramesPerSecond value
      frames.precision(2);
      frames.width(7);
      frames << "FPS: " << std::fixed << (float)mFrames / mFrameClock.GetElapsedTime();
      mFPS.SetText(frames.str());

      // Reset our Frames clock and frame counter
      mFrames = 0;
      mFrameClock.Reset();
    }

    // Are we showing the current statistics?
    if(mShow)
    {
      // Draw the Frames Per Second debug value on the screen
      mApp->mWindow.Draw(mFPS);

      // Draw the Updates Per Second debug value on the screen
      mApp->mWindow.Draw(mUPS);
    }
  }
}; // namespace GQE
```

## <a name="8" />IState.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the State class in the GQE namespace which is responsible for
 * providing the State interface used by all game engines.
 *
 * @file GQE/interfaces/IState.hpp
 * @author Ryan Lindeman
 * @date 20100705 - Initial Release
 * @date 20110125 - Use stringstream for UPS/FPS display
 * @date 20110128 - Moved UPS/FPS calculations to StatManager.
 * @date 20110131 - Added class and method argument documentation
 * @date 20110218 - Added ReInit method for StateManager::ResetActiveState
 * @date 20110218 - Reset our Cleanup flag after HandleCleanup is called and
 *                  call HandleCleanup in DoInit if cleanup flag is set.
 */
#ifndef   ISTATE_HPP_INCLUDED
#define   ISTATE_HPP_INCLUDED
 
#include <assert.h>
#include "GQE/GQE_types.hpp"
#include "GQE/classes/App.hpp"
#include <SFML/System.hpp>
 
namespace GQE
{
  /// Provides base class interface for all game states
  class IState
  {
  public:
 
    /**
     * State deconstructor
     */
    virtual ~IState() {
      if(NULL != mApp)
      {
        // Output to log file
        mApp->mLog << "IState::~IState() dtor with ID=" << mID << " was called" << std::endl;
      }
 
      // Clear out pointers that we don't need anymore
      mApp = NULL;
    }
 
    /**
     * GetID will return the ID used to identify this State object
     * @return GQE::typeStateID is the ID for this State object
     */
    const GQE::typeStateID GetID(void) const {
      return mID;
    }
 
    /**
     * DoInit is responsible for initializing this State.  HandleCleanup will
     * be called if mCleanup is true so Derived classes should always call
     * IState::DoInit() first before initializing their assets.
     */
    virtual void DoInit(void) {
      // Output to log file
      mApp->mLog << "IState::DoInit() with ID=" << mID << " was called" << std::endl;
      // If Cleanup hasn't been called yet, call it now!
      if(true == mCleanup)
      {
        HandleCleanup();
      }
      // Initialize if necessary
      if(false == mInit)
      {
        mInit = true;
        mPaused = false;
        mElapsedTime = 0.0f;
        mElapsedClock.Reset();
        mPausedTime = 0.0f;
        mPausedClock.Reset();
      }
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
    void DeInit(void) {
      mApp->mLog << "IState::DeInit() with ID=" << mID << " was called" << std::endl;
      if(true == mInit)
      {
        mCleanup = true;
        mInit = false;
        mElapsedTime += mElapsedClock.GetElapsedTime();
        if(true == mPaused)
        {
          mPausedTime += mPausedClock.GetElapsedTime();
        }
      }
    }
 
    /**
     * IsInitComplete will return true if the DoInit method has been called
     * for this state.
     * @return true if DoInit has been called, false otherwise or if DeInit
     *         was called
     */
    bool IsInitComplete(void) {
      return mInit;
    }
 
    /**
     * IsPaused will return true if this state is not the currently active
     * state.
     * @return true if state is not current active state, false otherwise
     */
    bool IsPaused(void) {
      return mPaused;
    }
 
    /**
     * Pause is responsible for pausing this State since the Application
     * may have lost focus or another State has become activate.
     */
    virtual void Pause(void) {
      if(NULL != mApp)
      {
        // Output to log file
        mApp->mLog << "State::Pause() with ID=" << mID << " was paused" << std::endl;
      }
 
      if(false == mPaused)
      {
        mPaused = true;
        mPausedClock.Reset();
      }
    }
 
    /**
     * Resume is responsible for resuming this State since the Application
     * may have gained focus or the previous State was removed.
     */
    virtual void Resume(void) {
      if(NULL != mApp)
      {
        // Output to log file
        mApp->mLog << "State::Resume() with ID=" << mID << " was resumed" << std::endl;
      }
 
      if(true == mPaused)
      {
        mPaused = false;
        mPausedTime += mPausedClock.GetElapsedTime();
      }
    }
 
    /**
     * HandleEvents is responsible for handling input events for this
     * State when it is the active State.
     * @param[in] theEvent to process from the App class Loop method
     */
    virtual void HandleEvents(sf::Event theEvent) = 0;
 
    /**
     * Update is responsible for handling all State update needs for this
     * State when it is the active State.
     */
    virtual void Update(void) = 0;
 
    /**
     * Draw is responsible for handling all Drawing needs for this State
     * when it is the Active State.
     */
    virtual void Draw(void) = 0;
 
    /**
     * HandleCleanup is responsible for calling Cleanup if this class has been
     * flagged to be cleaned up after it completes the game loop.
     */
    void HandleCleanup(void) {
      if(true == mCleanup)
      {
        // Call cleanup
        Cleanup();

        // Clear our cleanup flag
        mCleanup = false;
      }
    }
 
    /**
     * GetElapsedTime will return one of the following:
     * 1) If this state is not paused: total elapsed time since DoInit was called
     * 2) If this state is paused: total elapsed time since Pause was called
     * 3) If this state is not initialized: total elapsed time from DoInit to DeInit
     * @return total elapsed time as described above.
     */
    float GetElapsedTime(void) const {
      float result = mElapsedClock.GetElapsedTime();
 
      if(false == mInit)
      {
        result = mElapsedTime;
      }
 
      return result;
    }
 
  protected:
    /// Pointer to the App class
    App*                  mApp;
 
    /**
     * IState constructor is private because we do not allow copies of our
     * Singleton class
     * @param[in] theID to use for this State object
     * @param[in] theApp is a pointer to the App derived class
     */
    IState(const typeStateID theID, App* theApp) :
        mApp(theApp),
        mID(theID),
        mInit(false),
        mPaused(false),
        mCleanup(false),
        mElapsedTime(0.0f),
        mPausedTime(0.0f)
    {
      // Check that our pointer is good
      assert(NULL != theApp && "IState::IState() theApp pointer is bad");
 
      // Keep a copy of our Application pointer
      mApp = theApp;
 
      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "IState::IState() ctor with ID=" << mID << " was called" << std::endl;
      }
    }
 
    /**
     * Cleanup is responsible for performing any cleanup required before
     * this State is removed.
     */
    virtual void Cleanup(void) {
      // Output to log file
      mApp->mLog << "IState::Cleanup() with ID=" << mID << " was called" << std::endl;
    }
 
  private:
    /// The State ID
    const typeStateID     mID;
    /// Boolean that indicates that DoInit has been called
    bool                  mInit;
    /// State is currently paused (not active)
    bool                  mPaused;
    /// State needs to be cleaned up at the end of the next game loop
    bool                  mCleanup;
    /// Clock will help us keep track of running and paused elapsed time
    sf::Clock             mElapsedClock;
    /// Total elapsed time since DoInit was called
    float                 mElapsedTime;
    /// Clock will help us keep track of time paused
    sf::Clock             mPausedClock;
    /// Total elapsed time paused since DoInit was called
    float                 mPausedTime;
 
    /**
     * Our copy constructor is private because we do not allow copies of
     * our Singleton class
     */
    IState(const IState&);  // Intentionally undefined
 
    /**
     * Our assignment operator is private because we do not allow copies
     * of our Singleton class
     */
    IState& operator=(const IState&); // Intentionally undefined
 
  }; // class IState
}; // namespace GQE
 
#endif // ISTATE_HPP_INCLUDED
```

## <a name="9" />SplashState.hpp [ [Top] ](#top)
This is provided as an example of what a State would look like.  This Splash State loads the Splash.png image file and displays it to the screen for 10 seconds before asking the StateManager class to drop itself and move on to the next state which is the MenuState.

```cpp
/**
 * Provides the SplashState in the GQE namespace which is typically
 * the first state to be loaded for an application.
 *
 * @file GQE/states/SplashState.hpp
 * @author Ryan Lindeman
 * @date 20100710 - Initial Release
 * @date 20110131 - Added class and method argument documentation
 * @date 20110218 - Added ReInit method
 */
#ifndef   SPLASH_STATE_HPP_INCLUDED
#define   SPLASH_STATE_HPP_INCLUDED

#include "GQE/GQE_types.hpp" // Typedef declarations
#include "GQE/interfaces/IState.hpp"
#include <SFML/Graphics.hpp>

namespace GQE
{
  /// Provides simple Splash screen game state
  class SplashState : public IState
  {
  public:
    /**
     * SplashState constructor
     * @param[in] theApp is a pointer to the App class.
     */
    SplashState(App* theApp);

    /**
     * SplashState deconstructor
     */
    virtual ~SplashState(void);

    /**
     * DoInit is responsible for initializing this State
     */
    virtual void DoInit(void);

    /**
     * ReInit is responsible for Reseting this state when the 
     * StateManager::ResetActiveState() method is called.  This way a Game
     * State can be restarted without unloading and reloading the game assets
     */
    virtual void ReInit(void);

    /**
     * HandleEvents is responsible for handling input events for this
     * State when it is the active State.
     * @param[in] theEvent to process from the App class Loop method
     */
    virtual void HandleEvents(sf::Event theEvent);

    /**
     * Update is responsible for handling all State update needs for this
     * State when it is the active State.
     */
    virtual void Update(void);

    /**
     * Draw is responsible for handling all Drawing needs for this State
     * when it is the Active State.
     */
    virtual void Draw(void);

  protected:
    /**
     * Cleanup is responsible for performing any cleanup required before
     * this State is removed.
     */
    virtual void Cleanup(void);

  private:
    // Variables
    /////////////////////////////////////////////////////////////////////////
    sf::Sprite*         mSplashSprite;

  }; // class SplashState
}; // namespace GQE

#endif // SPLASH_STATE_HPP_INCLUDED
```

## <a name="10" />SplashState.cpp [ [Top] ](#top)
The implementation file for the SplashState
```cpp
/**
 * Provides the SplashState in the GQE namespace which is typically
 * the first state to be loaded for an application.
 *
 * @file GQE/states/SplashState.cpp
 * @author Ryan Lindeman
 * @date 20100710 - Initial Release
 * @date 20110125 - Use the new RemoveActiveState not DropActiveState.
 * @date 20110128 - Removed reference to IState::Update and IState::Draw
 * @date 20110218 - Added ReInit method
 */
#include "GQE/GQE_types.hpp" // Typedef declarations
#include "GQE/assets/ImageAsset.hpp"
#include "GQE/classes/App.hpp"
#include "GQE/states/SplashState.hpp"

namespace GQE
{
  SplashState::SplashState(App* theApp) :
    IState("Splash",theApp),
    mSplashSprite(NULL)
  {
  }

  SplashState::~SplashState(void)
  {
  }

  void SplashState::HandleEvents(sf::Event theEvent)
  {
  }

  void SplashState::DoInit(void)
  {
    // First call our base class implementation
    IState::DoInit();
    
    // Check our App pointer
    assert(NULL != mApp && "SplashState::DoInit() bad app pointer");

    // Load our splash image
    mApp->mAssetManager.AddImage("Splash", "SplashImage.png", AssetLoadStyleImmediate);

    // Retrieve a sprite to the above image
    mSplashSprite = mApp->mAssetManager.GetSprite("Splash");
  }

  void SplashState::ReInit(void)
  {
    // Do nothing yet
  }

  void SplashState::Update(void)
  {
    // Check our App pointer
    assert(NULL != mApp && "SplashState::Update() bad app pointer, init must be called first");

    // Drop our state after 10 seconds have elapsed
    if(false == IsPaused() && GetElapsedTime() > 10.0f)
    {
      mApp->mStateManager.RemoveActiveState();
    }
  }

  void SplashState::Draw(void)
  {
    // Check our App pointer
    assert(NULL != mApp && "SplashState::Draw() bad app pointer, init must be called first");

    // Draw our Splash sprite
    mApp->mWindow.Draw(*mSplashSprite);
  }

  void SplashState::Cleanup(void)
  {
    // Delete our sprite
    delete mSplashSprite;
    mSplashSprite = NULL;

    // Unload our image since we don't need it anymore
    mApp->mAssetManager.UnloadImage("Splash");

    // Last of all, call our base class implementation
    IState::Cleanup();
  }

}; // namespace GQE
```

## <a name="11" />MenuState.hpp [ [Top] ](#top)
This is provided as an example of what a Menu State would look like.  This Menu State loads the MenuBackground.png image file and displays it to the screen along with two Text menu options of "Start Game" and "Exit".  It accepts the "Escape" key as a signal to ask the StateManager class to drop itself and cause the Game Engine to exit since there will be no other Game States to switch to.

```cpp
/**
 * Provides the MenuState in the GQE namespace which is typically
 * the second state to be loaded for an application.
 *
 * @file GQE/states/MenuState.hpp
 * @author Ryan Lindeman
 * @date 20100717 - Initial Release
 * @date 20110131 - Added class and method argument documentation
 * @date 20110218 - Added ReInit method
 */
#ifndef   MENU_STATE_HPP_INCLUDED
#define   MENU_STATE_HPP_INCLUDED

#include "GQE/GQE_types.hpp" // Typedef declarations
#include "GQE/interfaces/IState.hpp"

namespace GQE
{
  /// Provides simple Menu game state
  class MenuState : public IState
  {
  public:
    /**
     * MenuState constructor
     * @param[in] theApp is a pointer to the App class.
     */
    MenuState(App* theApp);

    /**
     * MenuState deconstructor
     */
    virtual ~MenuState(void);

    /**
     * DoInit is responsible for initializing this State
     */
    virtual void DoInit(void);

    /**
     * ReInit is responsible for Reseting this state when the 
     * StateManager::ResetActiveState() method is called.  This way a Game
     * State can be restarted without unloading and reloading the game assets
     */
    virtual void ReInit(void);

    /**
     * HandleEvents is responsible for handling input events for this
     * State when it is the active State.
     * @param[in] theEvent to process from the App class Loop method
     */
    virtual void HandleEvents(sf::Event theEvent);

    /**
     * Update is responsible for handling all State update needs for this
     * State when it is the active State.
     */
    virtual void Update(void);

    /**
     * Draw is responsible for handling all Drawing needs for this State
     * when it is the Active State.
     */
    virtual void Draw(void);
  protected:
    /**
     * Cleanup is responsible for performing any cleanup required before
     * this State is removed.
     */
    virtual void Cleanup(void);

  private:
    // Variables
    /////////////////////////////////////////////////////////////////////////
    sf::Font*           mMenuFont;
    sf::Sprite*         mMenuSprite;
    sf::String*         mMenuString1;
    sf::String*         mMenuString2;

  }; // class MenuState
}; // namespace GQE

#endif // MENU_STATE_HPP_INCLUDED
```

## <a name="12" />MenuState.cpp [ [Top] ](#top)
The implementation file for the MenuState
```cpp
/**
 * Provides the MenuState in the GQE namespace which is typically
 * the first state to be loaded for an application.
 *
 * @file GQE/states/MenuState.cpp
 * @author Ryan Lindeman
 * @date 20100710 - Initial Release
 * @date 20110128 - Removed reference to IState::Update and IState::Draw
 * @date 20110218 - Added ReInit method
 */
#include "GQE/GQE_types.hpp" // Typedef declarations
#include "GQE/states/MenuState.hpp"
#include "GQE/classes/App.hpp"
#include "GQE/assets/FontAsset.hpp"

namespace GQE
{
  MenuState::MenuState(App* theApp) :
    IState("Menu", theApp),
    mMenuFont(NULL),
    mMenuSprite(NULL),
    mMenuString1(NULL),
    mMenuString2(NULL)
  {
  }

  MenuState::~MenuState(void)
  {
  }

  void MenuState::DoInit(void)
  {
    // First call our base class implementation
    IState::DoInit();
    
    // Check our App pointer
    assert(NULL != mApp && "MenuState::DoInit() bad app pointer");

    // Load our arial font
    mMenuFont = mApp->mAssetManager.AddFont("arial", "arial.ttf", AssetLoadStyleImmediate)->GetAsset();

    // Create our String
    mMenuString1 = new sf::String("Play Game", *mMenuFont);
    assert(NULL != mMenuString1 && "MenuState::DoInit() can't allocate memory for string");

    mMenuString2 = new sf::String("Exit", *mMenuFont);
    assert(NULL != mMenuString2 && "MenuState::DoInit() can't allocate memory for string");

    // Position the string
    mMenuString1->SetColor(sf::Color(255,255,255));
    mMenuString1->Move(400.f, 300.f);

    mMenuString2->SetColor(sf::Color(255,255,255));
    mMenuString2->Move(400.f, 400.f);

    // Load our splash image
    mApp->mAssetManager.AddImage("Menu", "MenuImage.png", AssetLoadStyleImmediate);

    // Retrieve a sprite to the above image
    mMenuSprite = mApp->mAssetManager.GetSprite("Menu");
  }

  void MenuState::ReInit(void)
  {
    // Do nothing yet
  }

  void MenuState::HandleEvents(sf::Event theEvent)
  {
    // Escape key pressed
    if ((theEvent.Type == sf::Event::KeyPressed) && (theEvent.Key.Code == sf::Key::Escape))
    {
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppOK);
      }
    }
  }

  void MenuState::Update(void)
  {
    // Check our App pointer
    assert(NULL != mApp && "MenuState::Update() bad app pointer, init must be called first");
  }

  void MenuState::Draw(void)
  {
    // Check our App pointer
    assert(NULL != mApp && "MenuState::Draw() bad app pointer, init must be called first");

    // Draw our Splash sprite
    mApp->mWindow.Draw(*mMenuSprite);

    // Draw our Menu String in the middle of the screen
    mApp->mWindow.Draw(*mMenuString2);
  }

  void MenuState::Cleanup(void)
  {
    // Delete our sprite
    delete mMenuSprite;
    mMenuSprite = NULL;

    // Delete our string
    delete mMenuString1;
    mMenuString1 = NULL;
    delete mMenuString2;
    mMenuString2 = NULL;

    // Unload our font since we don't need it anymore
    mApp->mAssetManager.UnloadFont("arial");

    // Unload our image since we don't need it anymore
    mApp->mAssetManager.UnloadImage("Menu");

    // Last of all, call our base class implementation
    IState::Cleanup();
  }

}; // namespace GQE
```

## <a name="13" />StateManager.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the StateManager class in the GQE namespace which is responsible
 * for providing the State management facilities to the App base class used by
 * all applications.
 *
 * @file GQE/classes/StateManager.hpp
 * @author Ryan Lindeman
 * @date 20100728 - Initial Release
 * @date 20110120 - Add ability to add inactive states
 * @date 20110120 - Add ability to drop active state as inactive state
 * @date 20110125 - IState::HandleCleanup is now called from here
 * @date 20110131 - Added class and method argument documentation
 * @date 20110218 - Change mDropped to mDead to remove potential confusion
 * @date 20110218 - Added InactivateActiveState and ResetActiveState methods
 */
#ifndef   STATE_MANAGER_HPP_INCLUDED
#define   STATE_MANAGER_HPP_INCLUDED
 
#include <vector>
#include <string>
#include "GQE/GQE_types.hpp"
 
namespace GQE
{
  /// Provides game state manager class for managing game states
  class StateManager
  {
  public:
    /**
     * StateManager constructor
     */
    StateManager();
 
    /**
     * StateManager deconstructor
     */
    virtual ~StateManager();
 
    /**
     * RegisterApp will register a pointer to the App class so it can be used
     * by the StateManager for error handling and log reporting.
     * @param[in] theApp is a pointer to the App (or App derived) class
     */
    void RegisterApp(App* theApp);
 
    /**
     * IsEmpty will return true if there are no active states on the stack.
     * @return true if state stack is empty, false otherwise.
     */
    bool IsEmpty(void);
 
    /**
     * AddActiveState will add theState provided as the current active
     * state.
     * @param[in] theState to set as the current state
     */
    void AddActiveState(IState* theState);
 
    /**
     * AddInactiveState will add theState provided as an inactive state.
     * @param[in] theState to be added as an inactive state
     */
    void AddInactiveState(IState* theState);

    /**
     * GetActiveState will return the current active state on the stack.
     * @return pointer to the current active state on the stack
     */
    IState* GetActiveState(void);

    /**
     * InactivateActiveState will cause the current active state to
     * become an inactive state without uninitializing its assets (doesn't
     * call DeInit) and return to the previous state on the stack. This
     * will cause the state to retain its assets.
     */
    void InactivateActivateState(void);

    /**
     * DropActiveState will cause the current active state to uninitialize
     * (calls DeInit) and become an inactive state and return to the
     * previous state on the stack. When a state is uninitialized its
     * assets are unloaded.
     */
    void DropActiveState(void);
 
    /**
     * ResetActiveState will cause the current active state to be reset
     * by calling the ReInit method for that state. This is useful for
     * "Play Again Y/N?" scenarios.
     */
    void ResetActiveState(void);

    /**
     * RemoveActiveState will cause the current active state to be removed
     * and return to the previous state on the stack.  Once a state has
     * been removed, you must re-add the state.  If you just want to
     * inactivate the current active state then use DropActiveState instead.
     */
    void RemoveActiveState(void);

    /**
     * SetActiveState will find the state specified by theStateID and make
     * it the current active state and move the previously current active
     * state as the next state.
     * @param[in] theStateID is the ID of the State to make active
     */
    void SetActiveState(typeStateID theStateID);
 
    /**
     * HandleCleanup is responsible for dealing with the cleanup of recently
     * dropped states.
     */
    void HandleCleanup(void);

  private:
    // Constants
    ///////////////////////////////////////////////////////////////////////////
 
    // Variables
    ///////////////////////////////////////////////////////////////////////////
    /// Pointer to the App class for error handling and logging
    App*                  mApp;
    /// Stack to store the current and previously active states
    std::vector<IState*>  mStack;
    /// Stack to store the dead states until they properly cleaned up
    std::vector<IState*>  mDead;
 
    /**
     * StateManager copy constructor is private because we do not allow copies
     * of our class
     */
    StateManager(const StateManager&); // Intentionally undefined
 
    /**
     * Our assignment operator is private because we do not allow copies
     * of our class
     */
    StateManager& operator=(const StateManager&); // Intentionally undefined
 
  }; // class StateManager
}; // namespace GQE
 
#endif // STATE_MANAGER_HPP_INCLUDED
```

## <a name="14" />StateManager.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the StateManager class in the GQE namespace which is responsible
 * for providing the State management facilities to the App base class used by
 * all applications.
 *
 * @file GQE/classes/StateManager.cpp
 * @author Ryan Lindeman
 * @date 20100728 - Initial Release
 * @date 20110120 - Add ability to add inactive states
 * @date 20110120 - Add ability to drop active state as inactive state
 * @date 20110125 - IState::HandleCleanup is now called from here
 * @date 20110218 - Change mDropped to mDead to remove potential confusion
 * @date 20110218 - Added InactivateActiveState and ResetActiveState methods
 */
 
#include <assert.h>
#include <stddef.h>
#include "GQE/classes/StateManager.hpp"
#include "GQE/classes/App.hpp"
#include "GQE/interfaces/IState.hpp"
 
namespace GQE
{
  StateManager::StateManager() :
    mApp(NULL)
  {
  }
 
  StateManager::~StateManager()
  {
    // Output to log file
    if(NULL != mApp)
    {
      mApp->mLog << "StateManager::~StateManager() dtor called" << std::endl;
    }
 
    // Drop all active states
    while(!mStack.empty())
    {
      // Retrieve the currently active state
      IState* anState = mStack.back();
 
      // Pop the currently active state off the stack
      mStack.pop_back();
 
      // Pause the currently active state
      anState->Pause();
 
      // De-initialize the state
      anState->DeInit();
 
      // Handle the cleanup before we pop it off the stack
      anState->HandleCleanup();
 
      // Just delete the state now
      delete anState;
 
      // Don't keep pointers around we don't need
      anState = NULL;
    }
 
    // Delete all our dropped states
    while(!mDead.empty())
    {
      // Retrieve the currently active state
      IState* anState = mDead.back();
 
      // Pop the currently active state off the stack
      mDead.pop_back();
 
      // Pause the currently active state
      anState->Pause();
 
      // De-initialize the state
      anState->DeInit();
 
      // Handle the cleanup before we pop it off the stack
      anState->HandleCleanup();
 
      // Just delete the state now
      delete anState;
 
      // Don't keep pointers around we don't need
      anState = NULL;
    }
 
    // Clear pointers we don't need anymore
    mApp = NULL;
  }
 
  void StateManager::RegisterApp(App* theApp)
  {
    // Check that our pointer is good
    assert(NULL != theApp && "StateManager::RegisterApp() theApp pointer provided is bad");
 
    // Make a note of the pointer
    assert(NULL == mApp && "StateManager::RegisterApp() theApp pointer was already registered");
    mApp = theApp;
  }
 
  bool StateManager::IsEmpty(void)
  {
    return mStack.empty();
  }
 
  void StateManager::AddActiveState(IState* theState)
  {
    // Check that they didn't provide a bad pointer
    assert(NULL != theState && "StateManager::AddActiveState() received a bad pointer");
 
    // Log the adding of each state
    if(NULL != mApp)
    {
      mApp->mLog << "StateManager::AddActiveState() StateID=" << theState->GetID() << std::endl;
    }
 
    // Is there a state currently running? then Pause it
    if(!mStack.empty())
    {
      // Pause the currently running state since we are changing the
      // currently active state to the one provided
      mStack.back()->Pause();
    }
 
    // Add the active state
    mStack.push_back(theState);
 
    // Initialize the new active state
    mStack.back()->DoInit();
  }
 
  void StateManager::AddInactiveState(IState* theState)
  {
    // Check that they didn't provide a bad pointer
    assert(NULL != theState && "StateManager::AddInactiveState() received a bad pointer");

    // Log the adding of each state
    if(NULL != mApp)
    {
      mApp->mLog << "StateManager::AddInactiveState() StateID=" << theState->GetID() << std::endl;
    }

    // Add the inactive state to the bottom of the stack
    mStack.insert(mStack.begin(), theState);
  }

  IState* StateManager::GetActiveState(void)
  {
    return mStack.back();
  }

  void StateManager::InactivateActivateState(void)
  {
    // Is there no currently active state to drop?
    if(!mStack.empty())
    {
      // Retrieve the currently active state
      IState* anState = mStack.back();
 
      // Log the dropping of each state
      if(NULL != mApp)
      {
        mApp->mLog << "StateManager::InactivateActiveState() StateID=" << anState->GetID() << std::endl;
      }
 
      // Pause the currently active state
      anState->Pause();

      // Pop the currently active state off the stack
      mStack.pop_back();
 
      // Move this now inactive state to the absolute back of our stack
      mStack.insert(mStack.begin(), anState);
 
      // Don't keep pointers around we don't need anymore
      anState = NULL;
    }
    else
    {
      // Quit the application with an error status response
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppStackEmpty);
      }
      return;
    }
 
    // Is there another state to activate? then call Resume to activate it
    if(!mStack.empty())
    {
      // Has this state ever been initialized?
      if(mStack.back()->IsInitComplete())
      {
        // Resume the new active state
        mStack.back()->Resume();
      }
      else
      {
        // Initialize the new active state
        mStack.back()->DoInit();
      }
    }
    else
    {
      // There are no states on the stack, exit the program
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppOK);
      }
    }
  }

  void StateManager::DropActiveState(void)
  {
    // Is there no currently active state to drop?
    if(!mStack.empty())
    {
      // Retrieve the currently active state
      IState* anState = mStack.back();
 
      // Log the dropping of each state
      if(NULL != mApp)
      {
        mApp->mLog << "StateManager::DropActiveState() StateID=" << anState->GetID() << std::endl;
      }
 
      // Pause the currently active state
      anState->Pause();

      // Deinit currently active state before we pop it off the stack
      // (HandleCleanup() will be called by IState::DoInit() method if this
      // state is ever set active again)
      anState->DeInit();
 
      // Pop the currently active state off the stack
      mStack.pop_back();
 
      // Move this now inactive state to the absolute back of our stack
      mStack.insert(mStack.begin(), anState);
 
      // Don't keep pointers around we don't need anymore
      anState = NULL;
    }
    else
    {
      // Quit the application with an error status response
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppStackEmpty);
      }
      return;
    }
 
    // Is there another state to activate? then call Resume to activate it
    if(!mStack.empty())
    {
      // Has this state ever been initialized?
      if(mStack.back()->IsInitComplete())
      {
        // Resume the new active state
        mStack.back()->Resume();
      }
      else
      {
        // Initialize the new active state
        mStack.back()->DoInit();
      }
    }
    else
    {
      // There are no states on the stack, exit the program
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppOK);
      }
    }
  }

  void StateManager::ResetActiveState(void)
  {
    // Is there no currently active state to reset?
    if(!mStack.empty())
    {
      // Retrieve the currently active state
      IState* anState = mStack.back();
 
      // Log the dropping of each state
      if(NULL != mApp)
      {
        mApp->mLog << "StateManager::ResetActiveState() StateID=" << anState->GetID() << std::endl;
      }
 
      // Pause the currently active state
      anState->Pause();
 
      // Call the ReInit method to Reset the currently active state
      anState->ReInit();
 
      // Resume the currently active state
      anState->Resume();
 
      // Don't keep pointers around we don't need anymore
      anState = NULL;
    }
    else
    {
      // Quit the application with an error status response
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppStackEmpty);
      }
      return;
    }
  }

  void StateManager::RemoveActiveState(void)
  {
    // Is there no currently active state to drop?
    if(!mStack.empty())
    {
      // Retrieve the currently active state
      IState* anState = mStack.back();
 
      // Log the dropping of each state
      if(NULL != mApp)
      {
        mApp->mLog << "StateManager::RemoveActiveState() StateID=" << anState->GetID() << std::endl;
      }
 
      // Pause the currently active state
      anState->Pause();
 
      // Cleanup the currently active state before we pop it off the stack
      anState->DeInit();
 
      // Pop the currently active state off the stack
      mStack.pop_back();
 
      // Move this state to our dropped stack
      mDead.push_back(anState);
 
      // Don't keep pointers around we don't need anymore
      anState = NULL;
    }
    else
    {
      // Quit the application with an error status response
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppStackEmpty);
      }
      return;
    }
 
    // Is there another state to activate? then call Resume to activate it
    if(!mStack.empty())
    {
      // Has this state ever been initialized?
      if(mStack.back()->IsInitComplete())
      {
        // Resume the new active state
        mStack.back()->Resume();
      }
      else
      {
        // Initialize the new active state
        mStack.back()->DoInit();
      }
    }
    else
    {
      // There are no states on the stack, exit the program
      if(NULL != mApp)
      {
        mApp->Quit(StatusAppOK);
      }
    }
  }
 
  void StateManager::SetActiveState(typeStateID theStateID)
  {
    std::vector<IState*>::iterator it;
 
    // Find the state that matches theStateID
    for(it=mStack.begin(); it < mStack.end(); it++)
    {
      // Does this state match theStateID? then activate it as the new
      // currently active state
      if((*it)->GetID() == theStateID)
      {
        // Get a pointer to soon to be currently active state
        IState* anState = *it;
 
        // Log the setting of a previously active state as the active state
        if(NULL != mApp)
        {
          mApp->mLog << "StateManager::SetActiveState() StateID=" << anState->GetID() << std::endl;
        }
 
        // Erase it from the list of previously active states
        mStack.erase(it);
 
        // Is there a state currently running? then Pause it
        if(!mStack.empty())
        {
          // Pause the currently running state since we are changing the
          // currently active state to the one specified by theStateID
          mStack.back()->Pause();
        }
 
        // Add the new active state
        mStack.push_back(anState);
 
        // Don't keep pointers we don't need around
        anState = NULL;
 
        // Has this state ever been initialized?
        if(mStack.back()->IsInitComplete())
        {
          // Resume the new active state
          mStack.back()->Resume();
        }
        else
        {
          // Initialize the new active state
          mStack.back()->DoInit();
        }
 
        // Exit our find loop
        break;
      } // if((*it)->GetID() == theStateID)
    } // for(it=mStack.begin(); it < mStack.end(); it++)
  }
 
  void StateManager::HandleCleanup(void)
  {
    // Remove one of our dead states
    if(!mDead.empty())
    {
      // Retrieve the dead state
      IState* anState = mDead.back();
      assert(NULL != anState && "StateManager::HandleCleanup() invalid dropped state pointer");
 
      // Pop the dead state off the stack
      mDead.pop_back();
 
      // Call the DeInit if it hasn't been called yet
      if(anState->IsInitComplete())
      {
        anState->DeInit();
      }

      // Next, call HandleCleanup for this state
      anState->HandleCleanup();

      // Just delete the state now
      delete anState;

      // Don't keep pointers around we don't need
      anState = NULL;
    }
  }

}; // namespace GQE
```

## <a name="15" />AssetManager.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the AssetManager class in the GQE namespace which is responsible
 * for providing the Asset management facilities to the App base class used by
 * all applications.
 *
 * @file GQE/classes/AssetManager.hpp
 * @author Ryan Lindeman
 * @date 20100723 - Initial Release
 * @date 20110131 - Added class and method argument documentation
 * @date 20110218 - Added new Config asset type
 */
#ifndef   ASSET_MANAGER_HPP_INCLUDED
#define   ASSET_MANAGER_HPP_INCLUDED

#include <map>
#include <string>
#include "GQE/GQE_types.hpp"
#include <SFML/Graphics.hpp>
#include <SFML/System.hpp>

namespace GQE
{
  /// Provides centralized game asset manager class for managing game assets.
  class AssetManager
  {
  public:
    /// Enumeration for all Asset Type values
    enum AssetType {
      FirstStandardAsset  = 0,  ///< First Standard Asset Type Value
      AssetConfig         = 1,  ///< Config file Asset Type
      AssetFont           = 2,  ///< Font Asset Type
      AssetImage          = 3,  ///< Image/Texture Asset Type
      AssetMusic          = 4,  ///< Background Music Asset Type
      AssetScript         = 5,  ///< Script Asset Type
      AssetSound          = 6,  ///< Sound Effect Asset Type
      AssetLevel          = 7,  ///< Level/Map Asset Type
      LastStandardAsset,        ///< Last Standard Asset Type Value

      // The following can be used for custom assets
      FirstCustomAsset    = 10, ///< First Custom Asset Type value
      AssetCustom1        = 11, ///< Custom Asset Type 1
      AssetCustom2        = 12, ///< Custom Asset Type 2
      AssetCustom3        = 13, ///< Custom Asset Type 3
      AssetCustom4        = 14, ///< Custom Asset Type 4
      AssetCustom5        = 15, ///< Custom Asset Type 5
      LastCustomAsset,          ///< Last Custom Asset Type Value
    };

    /**
     * AssetManager constructor
     */
    AssetManager();
 
    /**
     * AssetManager deconstructor
     */
    virtual ~AssetManager();
 
    /**
     * RegisterApp will register a pointer to the App class so it can be used
     * by the AssetManager for error handling and log reporting.
     * @param[in] theApp is a pointer to the App (or App derived) class
     */
    void RegisterApp(App* theApp);
 
    /**
     * LoadAssets will attempt to load all of the assets that have not been
     * loaded in either the foreground or background depending on
     * theBackgroundFlag provided.
     * @param[in] theBackgroundFlag means load assets in the background
     */
    void LoadAssets(bool theBackgroundFlag = false);
 
    /**
     * IsLoading() will return true if the background thread is currently
     * loading the Background loading style assets.
     * @return true if background thread is still running
     */
    bool IsLoading(void);

    /**
     * AddConfig will add a ConfigAsset object if the ConfigAsset object does
     * not yet exist, otherwise it will return a pointer to the existing
     * ConfigAsset.
     * @param[in] theAssetID is the ID for the asset to be added
     * @param[in] theFilename to use for loading this asset
     * @param[in] theStyle is the Loading style to use for this asset
     * @return pointer to the ConfigAsset that was added
     */
    ConfigAsset* AddConfig(
      const typeAssetID theAssetID,
      const std::string theFilename = "",
      AssetLoadingStyle theStyle = AssetLoadStyleBackground);

    /**
     * UnloadConfig will unload the Config asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the ConfigAsset to unload
     */
    void UnloadConfig(const typeAssetID theAssetID);

    /**
     * GetConfig will retrieve the Config asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the ConfigAsset to be retrieved
     * @return pointer to ConfigAsset or NULL if it doesn't yet exist
     */
    ConfigAsset* GetConfig(const typeAssetID theAssetID);

    /**
     * AddFont will add a FontAsset object if the FontAsset object does not
     * yet exist, otherwise it will return a pointer to the existing
     * FontAsset.
     * @param[in] theAssetID is the ID for the asset to be added
     * @param[in] theFilename to use for loading this asset
     * @param[in] theStyle is the Loading style to use for this asset
     * @return pointer to the FontAsset that was added
     */
    FontAsset* AddFont(
      const typeAssetID theAssetID,
      const std::string theFilename = "",
      AssetLoadingStyle theStyle = AssetLoadStyleBackground);
 
    /**
     * UnloadFont will unload the font asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the FontAsset to unload
     */
    void UnloadFont(const typeAssetID theAssetID);
 
    /**
     * GetFont will retrieve the font asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the FontAsset to be retrieved
     * @return pointer to FontAsset or NULL if it doesn't yet exist
     */
    FontAsset* GetFont(const typeAssetID theAssetID);
 
    /**
     * AddImage will add a ImageAsset object if the ImageAsset object does not
     * yet exist, otherwise it will return a pointer to the existing
     * ImageAsset.
     * @param[in] theAssetID is the ID for the asset to be added
     * @param[in] theFilename to use for loading this asset
     * @param[in] theStyle is the Loading style to use for this asset
     * @return pointer to the ImageAsset that was added
     */
    ImageAsset* AddImage(
      const typeAssetID theAssetID,
      const std::string theFilename = "",
      AssetLoadingStyle theStyle = AssetLoadStyleBackground);
 
    /**
     * GetImage will retrieve the image asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the ImageAsset to be retrieved
     * @return pointer to ImageAsset or NULL if it doesn't yet exist
     */
    ImageAsset* GetImage(const typeAssetID theAssetID);
 
    /**
     * GetSprite will return a sprite object for the image asset specified by
     * theAssetID.  The caller is responsible for this object and must delete
     * this object when he is through with it.
     * @param[in] theAssetID is the ID for the ImageAsset to be retrieved
     * @return pointer to sf::Sprite or NULL if image was not found
     */
    sf::Sprite* GetSprite(const typeAssetID theAssetID);
 
    /**
     * UnloadImage will unload the image asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the ImageAsset to unload
     */
    void UnloadImage(const typeAssetID theAssetID);
 
    /**
     * AddMusic will add a MusicAsset object if the MusicAsset object does not
     * yet exist, otherwise it will return a pointer to the existing
     * MusicAsset.
     * @param[in] theAssetID is the ID for the asset to be added
     * @param[in] theFilename to use for loading this asset
     * @param[in] theStyle is the Loading style to use for this asset
     * @return pointer to the MusicAsset that was added
     */
    MusicAsset* AddMusic(
      const typeAssetID theAssetID,
      const std::string theFilename = "",
      AssetLoadingStyle theStyle = AssetLoadStyleBackground);
 
    /**
     * GetMusic will retrieve the music asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the MusicAsset to be retrieved
     * @return pointer to MusicAsset or NULL if it doesn't yet exist
     */
    MusicAsset* GetMusic(const typeAssetID theAssetID);
 
    /**
     * UnloadMusic will unload the music asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the MusicAsset to unload
     */
    void UnloadMusic(const typeAssetID theAssetID);
 
    /**
     * AddSound will add a SoundAsset object if the SoundAsset object does not
     * yet exist, otherwise it will return a pointer to the existing
     * SoundAsset.
     * @param[in] theAssetID is the ID for the asset to be added
     * @param[in] theFilename to use for loading this asset
     * @param[in] theStyle is the Loading style to use for this asset
     * @return pointer to the FontAsset that was added
     */
    SoundAsset* AddSound(
      const typeAssetID theAssetID,
      const std::string theFilename = "",
      AssetLoadingStyle theStyle = AssetLoadStyleBackground);
 
    /**
     * GetSound will retrieve the sound asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the SoundAsset to be retrieved
     * @return pointer to SoundAsset or NULL if it doesn't yet exist
     */
    SoundAsset* GetSound(const typeAssetID theAssetID);
 
    /**
     * UnloadSound will unload the sound asset specified by theAssetID.
     * @param[in] theAssetID is the ID for the SoundAsset to unload
     */
    void UnloadSound(const typeAssetID theAssetID);

  private:
    // Constants
    ///////////////////////////////////////////////////////////////////////////
 
    // Variables
    ///////////////////////////////////////////////////////////////////////////
    /// Pointer to the App class for error handling and logging
    App*                                      mApp;
    /// Boolean indicating that the Background loading thread is running
    bool                                      mBackgroundLoading;
    /// Background loading thread
    sf::Thread*                               mBackgroundThread;
    /// Background loading thread mutex
    sf::Mutex                                 mBackgroundMutex;
    /// Map to store all the Config assets
    std::map<const typeAssetID, ConfigAsset*> mConfigs;
    /// Map to store all the Font assets
    std::map<const typeAssetID, FontAsset*>   mFonts;
    /// Map to store all the Image/Texture assets
    std::map<const typeAssetID, ImageAsset*>  mImages;
    /// Map to store all the Sound assets
    std::map<const typeAssetID, SoundAsset*>  mSounds;
    /// Map to store all the Music assets
    std::map<const typeAssetID, MusicAsset*>  mMusic;
 
    /**
     * AssetManager copy constructor is private because we do not allow copies
     * of our class
     */
    AssetManager(const AssetManager&); // Intentionally undefined
 
    /**
     * Our assignment operator is private because we do not allow copies
     * of our class
     */
    AssetManager& operator=(const AssetManager&); // Intentionally undefined
 
    /**
     * BackgroundLoop is the method that will be called when the background
     * thread is currently running.
     * @param[in] theAssetManager is a pointer to the AssetManager class
     */
    static void BackgroundLoop(void* theAssetManager);

    /**
     * DeleteConfigs will delete all added Config assets.
     */
    void DeleteConfigs(void);

    /**
     * LoadConfigs will load all the config that match theStyle specified.
     * @param[in] theStyle that equals the loading style of the unloaded assets
     */
    void LoadConfigs(AssetLoadingStyle theStyle);

    /**
     * DeleteFonts will delete all added font assets.
     */
    void DeleteFonts(void);
 
    /**
     * LoadFonts will load all the fonts that match theStyle specified.
     * @param[in] theStyle that equals the loading style of the unloaded assets
     */
    void LoadFonts(AssetLoadingStyle theStyle);
 
    /**
     * DeleteImages will delete all added image assets.
     */
    void DeleteImages(void);
 
    /**
     * LoadImages will load all the images that match theStyle specified.
     * @param[in] theStyle that equals the loading style of the unloaded assets
     */
    void LoadImages(AssetLoadingStyle theStyle);
 
    /**
     * DeleteMusic will delete all added music assets.
     */
    void DeleteMusic(void);
 
    /**
     * LoadMusic will load all the music that match theStyle specified.
     * @param[in] theStyle that equals the loading style of the unloaded assets
     */
    void LoadMusic(AssetLoadingStyle theStyle);
 
    /**
     * DeleteSound will delete all added sound assets.
     */
    void DeleteSounds(void);
 
    /**
     * LoadSounds will load all the sounds that match theStyle specified.
     * @param[in] theStyle that equals the loading style of the unloaded assets
     */
    void LoadSounds(AssetLoadingStyle theStyle);

  }; // class AssetManager
}; // namespace GQE

#endif // ASSET_MANAGER_HPP_INCLUDED
```

## <a name="16" />AssetManager.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the AssetManager class in the GQE namespace which is responsible
 * for providing the Asset management facilities to the App base class used by
 * all applications.
 *
 * @file GQE/classes/AssetManager.cpp
 * @author Ryan Lindeman
 * @date 20100723 - Initial Release
 * @date 20110128 - Fixed erase call in the DeleteXYZ methods.
 * @date 20110218 - Added ConfigAsset to AssetManager
 */

#include <assert.h>
#include <stddef.h>
#include "GQE/classes/AssetManager.hpp"
#include "GQE/classes/App.hpp"
#include "GQE/assets/ConfigAsset.hpp"
#include "GQE/assets/FontAsset.hpp"
#include "GQE/assets/ImageAsset.hpp"
#include "GQE/assets/MusicAsset.hpp"
#include "GQE/assets/SoundAsset.hpp"

namespace GQE
{
  AssetManager::AssetManager() :
    mApp(NULL),
    mBackgroundLoading(false),
    mBackgroundThread(NULL)
  {
    // Create our background loading thread for use later
    mBackgroundThread = new(std::nothrow) sf::Thread(&AssetManager::BackgroundLoop, this);
  }

  AssetManager::~AssetManager()
  {
    // Output to log file
    if(NULL != mApp)
    {
      mApp->mLog << "AssetManager::~AssetManager() dtor called" << std::endl;
    }

    if(true == mBackgroundLoading)
    {
      // Wait for thread to finish first
      mBackgroundThread->Wait();
    }

    // Cleanup after ourselves
    DeleteConfigs();
    DeleteFonts();
    DeleteImages();
    DeleteMusic();
    DeleteSounds();
    
    // Clear pointers we don't need anymore
    mApp = NULL;
  }

  void AssetManager::RegisterApp(App* theApp)
  {
    // Check that our pointer is good
    assert(NULL != theApp && "AssetManager::RegisterApp() theApp pointer provided is bad");

    // Make a note of the pointer
    assert(NULL == mApp && "AssetManager::RegisterApp() theApp pointer was already registered");
    mApp = theApp;
  }

  void AssetManager::LoadAssets(bool theBackgroundFlag)
  {
    // Background loading thread
    if(true == theBackgroundFlag && false == mBackgroundLoading)
    {
      // Output error condition to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::LoadAssets() starting background loading thread" << std::endl;
      }

      // Launch the background loading thread
      mBackgroundThread->Launch();
    }
    // Foreground loading thread
    else
    {
      // Output error condition to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::LoadAssets() starting foreground loading thread" << std::endl;
      }

      // Load all the configs
      LoadConfigs(AssetLoadStyleForeground);

      // Load all the fonts
      LoadFonts(AssetLoadStyleForeground);

      // Load all the images
      LoadImages(AssetLoadStyleForeground);

      // Load all the music
      LoadMusic(AssetLoadStyleForeground);

      // Load all the sounds
      LoadSounds(AssetLoadStyleForeground);
    }
  }

  bool AssetManager::IsLoading(void)
  {
    return mBackgroundLoading;
  }

  ConfigAsset* AssetManager::AddConfig(
    const typeAssetID theAssetID,
    const std::string theFilename,
    AssetLoadingStyle theStyle)
  {
    // ConfigAsset result
    ConfigAsset* result = NULL;

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Check for a duplicate asset using theAssetID as the key
    std::map<const typeAssetID, ConfigAsset*>::iterator iter;
    iter = mConfigs.find(theAssetID);
    if(iter == mConfigs.end())
    {
      // Create the asset since it wasn't found
      result = new (std::nothrow) ConfigAsset(theFilename, theStyle);
      assert(NULL != result && "AssetManager::AddConfig() unable to allocate memory");

      // Register our App pointer with this asset
      result->RegisterApp(mApp);

      // Handle Immediate loading style
      if(AssetLoadStyleImmediate == theStyle)
      {
        result->LoadAsset();
      }

      // Add the asset to the map of available assets
      mConfigs.insert(std::pair<const typeAssetID, ConfigAsset*>(theAssetID, result));
      
      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddConfig() adding asset with id=" << theAssetID << std::endl;
      }
    }
    else
    {
      // Return the previous asset that was added instead of creating a duplicate
      result = iter->second;

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddConfig() returning asset with id=" << theAssetID << std::endl;
      }
    }

    // Make sure we are actually returning a good pointer
    assert(NULL != result && "AssetManager::AddConfig() bad return result");

    // Increment our references to this asset
    result->AddReference();

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return our results
    return result;
  }

  void AssetManager::DeleteConfigs(void)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Delete all configs, since they are no longer needed
    std::map<const typeAssetID, ConfigAsset*>::iterator iter;
    iter = mConfigs.begin();
    while(iter != mConfigs.end())
    {
      ConfigAsset* anAsset = iter->second;
      mConfigs.erase(iter++);
      delete anAsset;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  ConfigAsset* AssetManager::GetConfig(const typeAssetID theAssetID)
  {
    ConfigAsset* result = NULL;
    
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, ConfigAsset*>::iterator iter;
    iter = mConfigs.find(theAssetID);
    if(iter != mConfigs.end())
    {
      // Get the asset found
      result = iter->second;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return the result found
    return result;
  }

  void AssetManager::LoadConfigs(AssetLoadingStyle theStyle)
  {
    assert(AssetLoadStyleFirst < theStyle && AssetLoadStyleLast > theStyle &&
           "AssetManager::LoadConfigs() invalid style provided");

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Load all unloaded configs with theStyle specified
    std::map<const typeAssetID, ConfigAsset*>::iterator iter;
    iter = mConfigs.begin();
    while(iter != mConfigs.end())
    {
      if(false == iter->second->IsLoaded() &&
         theStyle == iter->second->GetLoadingStyle())
      {
        iter->second->LoadAsset();
      }
      iter++;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  void AssetManager::UnloadConfig(const typeAssetID theAssetID)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, ConfigAsset*>::iterator iter;
    iter = mConfigs.find(theAssetID);
    if(iter != mConfigs.end())
    {
      // Drop our reference to this asset and unload if needed
      iter->second->DropReference();
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  FontAsset* AssetManager::AddFont(
    const typeAssetID theAssetID,
    const std::string theFilename,
    AssetLoadingStyle theStyle)
  {
    // FontAsset result
    FontAsset* result = NULL;

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Check for a duplicate asset using theAssetID as the key
    std::map<const typeAssetID, FontAsset*>::iterator iter;
    iter = mFonts.find(theAssetID);
    if(iter == mFonts.end())
    {
      // Create the asset since it wasn't found
      result = new(std::nothrow) FontAsset(theFilename, theStyle);
      assert(NULL != result && "AssetManager::AddFont() unable to allocate memory");

      // Register our App pointer with this asset
      result->RegisterApp(mApp);

      // Handle Immediate loading style
      if(AssetLoadStyleImmediate == theStyle)
      {
        result->LoadAsset();
      }

      // Add the asset to the map of available assets
      mFonts.insert(std::pair<const typeAssetID, FontAsset*>(theAssetID, result));
      
      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddFont() adding asset with id=" << theAssetID << std::endl;
      }
    }
    else
    {
      // Return the previous asset that was added instead of creating a duplicate
      result = iter->second;

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddFont() returning asset with id=" << theAssetID << std::endl;
      }
    }

    // Make sure we are actually returning a good pointer
    assert(NULL != result && "AssetManager::AddFont() bad return result");

    // Increment our references to this asset
    result->AddReference();

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return our results
    return result;
  }

  void AssetManager::DeleteFonts(void)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Delete all fonts, since they are no longer needed
    std::map<const typeAssetID, FontAsset*>::iterator iter;
    iter = mFonts.begin();
    while(iter != mFonts.end())
    {
      FontAsset* anAsset = iter->second;
      mFonts.erase(iter++);
      delete anAsset;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  FontAsset* AssetManager::GetFont(const typeAssetID theAssetID)
  {
    FontAsset* result = NULL;
    
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, FontAsset*>::iterator iter;
    iter = mFonts.find(theAssetID);
    if(iter != mFonts.end())
    {
      // Get the asset found
      result = iter->second;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return the result found
    return result;
  }

  void AssetManager::LoadFonts(AssetLoadingStyle theStyle)
  {
    assert(AssetLoadStyleFirst < theStyle && AssetLoadStyleLast > theStyle &&
           "AssetManager::LoadFonts() invalid style provided");

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Load all unloaded fonts with theStyle specified
    std::map<const typeAssetID, FontAsset*>::iterator iter;
    iter = mFonts.begin();
    while(iter != mFonts.end())
    {
      if(false == iter->second->IsLoaded() &&
         theStyle == iter->second->GetLoadingStyle())
      {
        iter->second->LoadAsset();
      }
      iter++;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  void AssetManager::UnloadFont(const typeAssetID theAssetID)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, FontAsset*>::iterator iter;
    iter = mFonts.find(theAssetID);
    if(iter != mFonts.end())
    {
      // Drop our reference to this asset and unload if needed
      iter->second->DropReference();
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  ImageAsset* AssetManager::AddImage(
    const typeAssetID theAssetID,
    const std::string theFilename,
    AssetLoadingStyle theStyle)
  {
    // Our return result
    ImageAsset* result = NULL;

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Check for a duplicate asset using theAssetID as the key
    std::map<const typeAssetID, ImageAsset*>::iterator iter;
    iter = mImages.find(theAssetID);
    if(iter == mImages.end())
    {
      // Create the asset since it wasn't found
      result = new(std::nothrow) ImageAsset(theFilename, theStyle);
      assert(NULL != result && "AssetManager::AddImage() unable to allocate memory");

      // Register our App pointer with this asset
      result->RegisterApp(mApp);

      // Handle Immediate loading style
      if(AssetLoadStyleImmediate == theStyle)
      {
        result->LoadAsset();
      }

      // Add the asset to the map of available assets
      mImages.insert(std::pair<const typeAssetID, ImageAsset*>(theAssetID, result));
      
      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddImage() adding asset with id=" << theAssetID << std::endl;
      }
    }
    else
    {
      // Return the previous asset that was added instead of creating a duplicate
      result = iter->second;

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddImage() returning asset with id=" << theAssetID << std::endl;
      }
    }

    // Make sure we are actually returning a good pointer
    assert(NULL != result && "AssetManager::AddImage() bad return result");

    // Increment our references to this asset
    result->AddReference();

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return our results
    return result;
  }

  void AssetManager::DeleteImages(void)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Delete all images, since they are no longer needed
    std::map<const typeAssetID, ImageAsset*>::iterator iter;
    iter = mImages.begin();
    while(iter != mImages.end())
    {
      ImageAsset* anAsset = iter->second;
      mImages.erase(iter++);
      delete anAsset;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  ImageAsset* AssetManager::GetImage(const typeAssetID theAssetID)
  {
    ImageAsset* result = NULL;
    
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, ImageAsset*>::iterator iter;
    iter = mImages.find(theAssetID);
    if(iter != mImages.end())
    {
      // Get the asset found
      result = iter->second;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return the result found
    return result;
  }

  /**
   * GetSprite will return a sprite object for the image asset specified by
   * theAssetID.  The caller is responsible for this object and must delete
   * this object when he is through with it.
   * @param[in] theAssetID is the ID for the ImageAsset to be retrieved
   * @return pointer to sf::Sprite or NULL if image was not found
   */
  sf::Sprite* AssetManager::GetSprite(const typeAssetID theAssetID)
  {
    sf::Sprite* result = NULL;
    ImageAsset* anImageAsset = GetImage(theAssetID);
    if(NULL != anImageAsset)
    {
      sf::Image* anImage = anImageAsset->GetAsset();
      assert(NULL != anImage && "AssetManager::GetSprite() failed to obtain image asset");
      result = new(std::nothrow) sf::Sprite(*anImage);
      assert(NULL != result && "AssetManager::GetSprite() unable to allocate memory");
    }
    else
    {
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::GetSprite() failed to find image with id=" << theAssetID << std::endl;
        mApp->Quit(StatusAppMissingAsset);
      }
    }

    // Return our result
    return result;
  }

  void AssetManager::LoadImages(AssetLoadingStyle theStyle)
  {
    assert(AssetLoadStyleFirst < theStyle && AssetLoadStyleLast > theStyle &&
           "AssetManager::LoadImages() invalid style provided");

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Load all unloaded images with theStyle specified
    std::map<const typeAssetID, ImageAsset*>::iterator iter;
    iter = mImages.begin();
    while(iter != mImages.end())
    {
      if(false == iter->second->IsLoaded() &&
         theStyle == iter->second->GetLoadingStyle())
      {
        iter->second->LoadAsset();
      }
      iter++;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  void AssetManager::UnloadImage(const typeAssetID theAssetID)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, ImageAsset*>::iterator iter;
    iter = mImages.find(theAssetID);
    if(iter != mImages.end())
    {
      // Drop our reference to this asset and unload if needed
      iter->second->DropReference();
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  MusicAsset* AssetManager::AddMusic(
    const typeAssetID theAssetID,
    const std::string theFilename,
    AssetLoadingStyle theStyle)
  {
    // Our return result
    MusicAsset* result = NULL;

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Check for a duplicate asset using theAssetID as the key
    std::map<const typeAssetID, MusicAsset*>::iterator iter;
    iter = mMusic.find(theAssetID);
    if(iter == mMusic.end())
    {
      // Create the asset since it wasn't found
      result = new(std::nothrow) MusicAsset(theFilename, theStyle);
      assert(NULL != result && "AssetManager::AddMusic() unable to allocate memory");

      // Register our App pointer with this asset
      result->RegisterApp(mApp);

      // Handle Immediate loading style
      if(AssetLoadStyleImmediate == theStyle)
      {
        result->LoadAsset();
      }

      // Add the asset to the map of available assets
      mMusic.insert(std::pair<const typeAssetID, MusicAsset*>(theAssetID, result));
      
      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddMusic() adding asset with id=" << theAssetID << std::endl;
      }
    }
    else
    {
      // Return the previous asset that was added instead of creating a duplicate
      result = iter->second;

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddMusic() returning asset with id=" << theAssetID << std::endl;
      }
    }

    // Make sure we are actually returning a good pointer
    assert(NULL != result && "AssetManager::AddMusic() bad return result");

    // Increment our references to this asset
    result->AddReference();

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return our results
    return result;
  }

  void AssetManager::DeleteMusic(void)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Delete all music, since they are no longer needed
    std::map<const typeAssetID, MusicAsset*>::iterator iter;
    iter = mMusic.begin();
    while(iter != mMusic.end())
    {
      MusicAsset* anAsset = iter->second;
      mMusic.erase(iter++);
      delete anAsset;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  MusicAsset* AssetManager::GetMusic(const typeAssetID theAssetID)
  {
    MusicAsset* result = NULL;
    
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, MusicAsset*>::iterator iter;
    iter = mMusic.find(theAssetID);
    if(iter != mMusic.end())
    {
      // Get the asset found
      result = iter->second;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return the result found
    return result;
  }

  void AssetManager::LoadMusic(AssetLoadingStyle theStyle)
  {
    assert(AssetLoadStyleFirst < theStyle && AssetLoadStyleLast > theStyle &&
           "AssetManager::LoadMusic() invalid style provided");

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Load all unloaded music with theStyle specified
    std::map<const typeAssetID, MusicAsset*>::iterator iter;
    iter = mMusic.begin();
    while(iter != mMusic.end())
    {
      if(false == iter->second->IsLoaded() &&
         theStyle == iter->second->GetLoadingStyle())
      {
        iter->second->LoadAsset();
      }
      iter++;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  void AssetManager::UnloadMusic(const typeAssetID theAssetID)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, MusicAsset*>::iterator iter;
    iter = mMusic.find(theAssetID);
    if(iter != mMusic.end())
    {
      // Drop our reference to this asset and unload if needed
      iter->second->DropReference();
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  SoundAsset* AssetManager::AddSound(
    const typeAssetID theAssetID,
    const std::string theFilename,
    AssetLoadingStyle theStyle)
  {
    // Our return result
    SoundAsset* result = NULL;

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Check for a duplicate asset using theAssetID as the key
    std::map<const typeAssetID, SoundAsset*>::iterator iter;
    iter = mSounds.find(theAssetID);
    if(iter == mSounds.end())
    {
      // Create the asset since it wasn't found
      result = new(std::nothrow) SoundAsset(theFilename, theStyle);
      assert(NULL != result && "AssetManager::AddSound() unable to allocate memory");

      // Register our App pointer with this asset
      result->RegisterApp(mApp);

      // Handle Immediate loading style
      if(AssetLoadStyleImmediate == theStyle)
      {
        result->LoadAsset();
      }

      // Add the asset to the map of available assets
      mSounds.insert(std::pair<const typeAssetID, SoundAsset*>(theAssetID, result));
      
      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddSound() adding asset with id=" << theAssetID << std::endl;
      }
    }
    else
    {
      // Return the previous asset that was added instead of creating a duplicate
      result = iter->second;

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "AssetManager::AddSound() returning asset with id=" << theAssetID << std::endl;
      }
    }

    // Make sure we are actually returning a good pointer
    assert(NULL != result && "AssetManager::AddSound() bad return result");

    // Increment our references to this asset
    result->AddReference();

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return our results
    return result;
  }

  void AssetManager::DeleteSounds(void)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Delete all sounds, since they are no longer needed
    std::map<const typeAssetID, SoundAsset*>::iterator iter;
    iter = mSounds.begin();
    while(iter != mSounds.end())
    {
      SoundAsset* anAsset = iter->second;
      mSounds.erase(iter++);
      delete anAsset;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  SoundAsset* AssetManager::GetSound(const typeAssetID theAssetID)
  {
    SoundAsset* result = NULL;
    
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, SoundAsset*>::iterator iter;
    iter = mSounds.find(theAssetID);
    if(iter != mSounds.end())
    {
      // Get the asset found
      result = iter->second;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();

    // Return the result found
    return result;
  }

  void AssetManager::LoadSounds(AssetLoadingStyle theStyle)
  {
    assert(AssetLoadStyleFirst < theStyle && AssetLoadStyleLast > theStyle &&
           "AssetManager::LoadSounds() invalid style provided");

    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Load all unloaded sounds with theStyle specified
    std::map<const typeAssetID, SoundAsset*>::iterator iter;
    iter = mSounds.begin();
    while(iter != mSounds.end())
    {
      if(false == iter->second->IsLoaded() &&
         theStyle == iter->second->GetLoadingStyle())
      {
        iter->second->LoadAsset();
      }
      iter++;
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  void AssetManager::UnloadSound(const typeAssetID theAssetID)
  {
    // Obtain a lock before trying to access the asset maps
    mBackgroundMutex.Lock();

    // Locate the asset using theAssetID as the key
    std::map<const typeAssetID, SoundAsset*>::iterator iter;
    iter = mSounds.find(theAssetID);
    if(iter != mSounds.end())
    {
      // Drop our reference to this asset and unload if needed
      iter->second->DropReference();
    }

    // Release the lock after accessing the asset maps
    mBackgroundMutex.Unlock();
  }

  void AssetManager::BackgroundLoop(void* theAssetManager)
  {
    AssetManager* anManager = static_cast<AssetManager*>(theAssetManager);
    assert(NULL != anManager && "AssetManager::BackgroundLoop() invalid pointer provided");

    // Set our running flag
    anManager->mBackgroundLoading = true;

    // Load all the configs
    anManager->LoadConfigs(AssetLoadStyleBackground);

    // Load all the fonts
    anManager->LoadFonts(AssetLoadStyleBackground);

    // Load all the images
    anManager->LoadImages(AssetLoadStyleBackground);

    // Load all the music
    anManager->LoadMusic(AssetLoadStyleBackground);

    // Load all the sounds
    anManager->LoadSounds(AssetLoadStyleBackground);

    // Set our running flag
    anManager->mBackgroundLoading = false;
  }
}; // namespace GQE
```

## <a name="17" />TAsset.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the TAsset template class for managing a single Asset and is used
 * by the AssetManager in the GQE namespace which is responsible for providing
 * the Asset management facilities for the App base class used by all
 * applications.
 *
 * @file GQE/interfaces/TAsset.hpp
 * @author Ryan Lindeman
 * @date 20100723 - Initial Release
 */
#ifndef   TASSET_HPP_INCLUDED
#define   TASSET_HPP_INCLUDED

#include <assert.h>
#include <string>
#include <stddef.h>
#include <stdint.h>
#include "GQE/GQE_types.hpp"

namespace GQE
{
  template<class TYPE>
  class TAsset
  {
  public:
    /**
     * TAsset constructor
     * @param[in] theFilename to use when loading this asset
     * @param[in] theStyle to use when loading this asset
     */
    TAsset(std::string theFilename, AssetLoadingStyle theStyle = AssetLoadStyleBackground) :
      mApp(NULL),
      mFilename(theFilename),
      mStyle(theStyle),
      mAsset(NULL),
      mLoaded(false),
      mReferences(0)
    {
      assert(AssetLoadStyleFirst < theStyle && AssetLoadStyleLast > theStyle &&
             "TAsset::TAsset() invalid style provided");
    }

    /**
     * TAsset deconstructor
     */
    virtual ~TAsset() {
      if(0 != mReferences)
      {
        // Log an error, since not all of our assets have had their FreeAsset
        // method called!
      }
    }

    /**
     * RegisterApp will register a pointer to the App class so it can be used
     * by the TAsset derived classes for error handling and log reporting.
     * @param[in] theApp is a pointer to the App (or App derived) class
     */
    void RegisterApp(App* theApp) {
      // Check that our pointer is good
      assert(NULL != theApp && "TAsset::RegisterApp() theApp pointer provided is bad");

      // Make a note of the pointer
      assert(NULL == mApp && "TAsset::RegisterApp() theApp pointer was already registered");
      mApp = theApp;
    }

    /**
     * IsLoaded will return true if the Asset has been loaded.
     * @return true if loaded, false otherwise
     */
    bool IsLoaded(void) const {
      return mLoaded;
    }

    /**
     * GetLoadingStyle will return the Loading Style for this asset.
     * @return LoadingStyle enumeration for this asset
     */
    AssetLoadingStyle GetLoadingStyle(void) const {
      return mStyle;
    }

    /**
     * SetLoadingStyle will set the Loading Style for this asset.
     * @param[in] theStyle to use when Loading this asset
     */
    void SetLoadingStyle(AssetLoadingStyle theStyle) {
      assert(AssetLoadStyleFirst < theStyle && AssetLoadStyleLast > theStyle &&
             "TAsset::SetLoadingStyle() invalid style provided");

      mStyle = theStyle;
    }

    /**
     * AddReference will increment the reference count.
     */
    void AddReference(void)
    {
      mReferences++;
    }

    /**
     * GetReferences will return the reference count.
     * @return number of entities referencing this Asset
     */
    const uint16_t GetReferences(void) const {
      return mReferences;
    }

    /**
     * DropReference will decrement the reference count.
     */
    void DropReference(bool theRemoveFlag = true) {
      assert(0 != mReferences && "TAsset::DropReference() called more than AddReference()");
      mReferences--;
      
      // Delete asset for zero references and theRemoveFlag is set
      if(true == theRemoveFlag && 0 == mReferences)
      {
        UnloadAsset();
      }
    }

    /**
     * GetAsset will return the Asset if it is available.
     * @return pointer to the Asset or NULL if not available yet.
     */
    TYPE* GetAsset(void) const {
      return mAsset;
    }

    /**
     * LoadAsset is responsible for loading the Asset.
     */
    virtual void LoadAsset(void) = 0;

  protected:
    // Variables
    ///////////////////////////////////////////////////////////////////////////
    /// Pointer to the App class for error handling and logging
    App*                mApp;
    /// The filename associated with this asset
    const std::string   mFilename;
    /// The loading style to use with this asset
    AssetLoadingStyle   mStyle;
    /// Pointer to the loaded asset
    TYPE*               mAsset;
    /// True if the asset has been loaded or provided
    bool                mLoaded;

    /**
     * UnloadAsset is responsible for destroying or unloading the Asset and
     * is called by FreeAsset.
     */
    virtual void UnloadAsset(void) = 0;

  private:
    /// A counter for every time this asset is referenced
    uint16_t    mReferences;

    /**
     * Our copy constructor is private because we do not allow copies of our
     * class
     */
    TAsset(const TAsset&);  // Intentionally undefined

    /**
     * Our assignment operator is private because we do not allow copies of our
     * class
     */
    TAsset& operator=(const TAsset&); // Intentionally undefined
  }; // class TAsset
}; // namespace GQE

#endif // TASSET_HPP_INCLUDED
```

## <a name="18" />ConfigAsset.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the Config Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file include/GQE/assets/ConfigAsset.hpp
 * @author Ryan Lindeman
 * @date 20110218 - Initial Release
 */
#ifndef   CONFIG_ASSET_HPP_INCLUDED
#define   CONFIG_ASSET_HPP_INCLUDED

#include "GQE/interfaces/TAsset.hpp"
#include "GQE/GQE_types.hpp"
#include "GQE/classes/ConfigReader.hpp"

namespace GQE
{
  class ConfigAsset : public TAsset<ConfigReader>
  {
  public:
    /**
     * ConfigAsset constructor
     * @param[in] theFilename to use when loading this asset
     * @param[in] theStyle to use when loading this asset
     */
    ConfigAsset(std::string theFilename, AssetLoadingStyle theStyle = AssetLoadStyleBackground);

    /**
     * ConfigAsset deconstructor
     */
    virtual ~ConfigAsset();

    /**
     * LoadAsset is responsible for loading the Asset.
     */
    virtual void LoadAsset(void);

  protected:
    /**
     * UnloadAsset is responsible for destroying or unloading the Asset and
     * is called by FreeAsset.
     */
    virtual void UnloadAsset(void);

  private:
    // Variables
    ///////////////////////////////////////////////////////////////////////////
  }; // class ConfigAsset
}; // namespace GQE

#endif // CONFIG_ASSET_HPP_INCLUDED
```

## <a name="19" />ConfigAsset.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the Config Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/ConfigAsset.cpp
 * @author Ryan Lindeman
 * @date 20110218 - Initial Release
 */

#include <assert.h>
#include <stddef.h>
#include "GQE/assets/ConfigAsset.hpp"
#include "GQE/classes/App.hpp"

namespace GQE
{
  ConfigAsset::ConfigAsset(std::string theFilename, AssetLoadingStyle theStyle) :
    TAsset<ConfigReader>(theFilename, theStyle)
  {
  }

  ConfigAsset::~ConfigAsset()
  {
    UnloadAsset();
  }

  void ConfigAsset::LoadAsset(void)
  {
    // Only load the asset once if possible!
    if(false == mLoaded)
    {
      // Make sure memory is not already allocated
      assert(NULL == mAsset && "ConfigAsset::LoadAsset() memory already allocated!");
    
      // Create the asset
      mAsset = new (std::nothrow) ConfigReader;
      assert(NULL != mAsset && "ConfigAsset::LoadAsset() unable to allocate memory");

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "ConfigAsset::LoadAsset() loading asset with filename=" << mFilename << std::endl;
      }

      // Attempt to load the asset from a file
      mLoaded = mAsset->Read(mFilename);

      // Register the App pointer with this asset
      mAsset->RegisterApp(mApp);

      // If the asset did not load successfully, delete the memory
      if(false == mLoaded)
      {
        // Output to log file
        if(NULL != mApp)
        {
          mApp->mLog << "ConfigAsset::LoadAsset() asset with filename=" << mFilename << " is missing" << std::endl;
          mApp->Quit(StatusAppMissingAsset);
        }
      }
    }
  }

  void ConfigAsset::UnloadAsset(void)
  {
    // Delete the asset, forcing it to be removed from memory
    delete mAsset;
    mAsset = NULL;
    mLoaded = false;
  }

}; // namespace GQE
```

## <a name="20" />FontAsset.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the Font Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/FontAsset.hpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */
#ifndef   FONT_ASSET_HPP_INCLUDED
#define   FONT_ASSET_HPP_INCLUDED

#include "GQE/interfaces/TAsset.hpp"
#include "GQE/GQE_types.hpp"
#include <SFML/Graphics.hpp>

namespace GQE
{
  class FontAsset : public TAsset<sf::Font>
  {
  public:
    /**
     * FontAsset constructor
     * @param[in] theFilename to use when loading this asset
     * @param[in] theStyle to use when loading this asset
     */
    FontAsset(std::string theFilename, AssetLoadingStyle theStyle = AssetLoadStyleBackground);

    /**
     * FontAsset deconstructor
     */
    virtual ~FontAsset();

    /**
     * LoadAsset is responsible for loading the Asset.
     */
    virtual void LoadAsset(void);

  protected:
    /**
     * UnloadAsset is responsible for destroying or unloading the Asset and
     * is called by FreeAsset.
     */
    virtual void UnloadAsset(void);

  private:
    // Variables
    ///////////////////////////////////////////////////////////////////////////
  }; // class FontAsset
}; // namespace GQE

#endif // FONT_ASSET_HPP_INCLUDED
```

## <a name="21" />FontAsset.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the Font Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/FontAsset.cpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */

#include <assert.h>
#include <stddef.h>
#include "GQE/assets/FontAsset.hpp"
#include "GQE/classes/App.hpp"

namespace GQE
{
  FontAsset::FontAsset(std::string theFilename, AssetLoadingStyle theStyle) :
    TAsset<sf::Font>(theFilename, theStyle)
  {
  }

  FontAsset::~FontAsset()
  {
    UnloadAsset();
  }

  void FontAsset::LoadAsset(void)
  {
    // Only load the asset once if possible!
    if(false == mLoaded)
    {
      // Make sure memory is not already allocated
      assert(NULL == mAsset && "FontAsset::LoadAsset() memory already allocated!");
    
      // Create the asset
      mAsset = new(std::nothrow) sf::Font;
      assert(NULL != mAsset && "FontAsset::LoadAsset() unable to allocate memory");

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "FontAsset::LoadAsset() loading asset with filename=" << mFilename << std::endl;
      }

      // Attempt to load the asset from a file
      mLoaded = mAsset->LoadFromFile(mFilename);

      // If the asset did not load successfully, delete the memory
      if(false == mLoaded)
      {
        // Output to log file
        if(NULL != mApp)
        {
          mApp->mLog << "FontAsset::LoadAsset() asset with filename=" << mFilename << " is missing" << std::endl;
          mApp->Quit(StatusAppMissingAsset);
        }
      }
    }
  }

  void FontAsset::UnloadAsset(void)
  {
    // Delete the asset, forcing it to be removed from memory
    delete mAsset;
    mAsset = NULL;
    mLoaded = false;
  }

}; // namespace GQE
```

## <a name="22" />ImageAsset.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the Image Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/ImageAsset.hpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */
#ifndef   IMAGE_ASSET_HPP_INCLUDED
#define   IMAGE_ASSET_HPP_INCLUDED

#include "GQE/interfaces/TAsset.hpp"
#include "GQE/GQE_types.hpp"
#include <SFML/Graphics.hpp>

namespace GQE
{
  class ImageAsset : public TAsset<sf::Image>
  {
  public:
    /**
     * ImageAsset constructor
     * @param[in] theFilename to use when loading this asset
     * @param[in] theStyle to use when loading this asset
     */
    ImageAsset(std::string theFilename, AssetLoadingStyle theStyle = AssetLoadStyleBackground);

    /**
     * ImageAsset deconstructor
     */
    virtual ~ImageAsset();

    /**
     * LoadAsset is responsible for loading the Asset.
     */
    virtual void LoadAsset(void);

  protected:
    /**
     * UnloadAsset is responsible for destroying or unloading the Asset and
     * is called by FreeAsset.
     */
    virtual void UnloadAsset(void);

  private:
    // Variables
    ///////////////////////////////////////////////////////////////////////////
  }; // class ImageAsset
}; // namespace GQE

#endif // IMAGE_ASSET_HPP_INCLUDED
```

## <a name="23" />ImageAsset.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the Image Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/ImageAsset.cpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */

#include <assert.h>
#include <stddef.h>
#include "GQE/assets/ImageAsset.hpp"
#include "GQE/classes/App.hpp"

namespace GQE
{
  ImageAsset::ImageAsset(std::string theFilename, AssetLoadingStyle theStyle) :
    TAsset<sf::Image>(theFilename, theStyle)
  {
  }

  ImageAsset::~ImageAsset()
  {
    UnloadAsset();
  }

  void ImageAsset::LoadAsset(void)
  {
    // Only load the asset once if possible!
    if(false == mLoaded)
    {
      // Make sure memory is not already allocated
      assert(NULL == mAsset && "ImageAsset::LoadAsset() memory already allocated!");
    
      // Create the asset
      mAsset = new(std::nothrow) sf::Image;
      assert(NULL != mAsset && "ImageAsset::LoadAsset() unable to allocate memory");

      // Disable smooth of this image to allow for pixel perfect display
      mAsset->SetSmooth(false);

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "ImageAsset::LoadAsset() loading asset with filename=" << mFilename << std::endl;
      }

      // Attempt to load the asset from a file
      mLoaded = mAsset->LoadFromFile(mFilename);

      // If the asset did not load successfully, delete the memory
      if(false == mLoaded)
      {
        // Output to log file
        if(NULL != mApp)
        {
          mApp->mLog << "ImageAsset::LoadAsset() asset with filename=" << mFilename << " is missing" << std::endl;
          mApp->Quit(StatusAppMissingAsset);
        }
      }
    }
  }

  void ImageAsset::UnloadAsset(void)
  {
    // Delete the asset, forcing it to be removed from memory
    delete mAsset;
    mAsset = NULL;
    mLoaded = false;
  }

}; // namespace GQE
```

## <a name="24" />MusicAsset.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the Music Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/MusicAsset.hpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */
#ifndef   MUSIC_ASSET_HPP_INCLUDED
#define   MUSIC_ASSET_HPP_INCLUDED

#include "GQE/interfaces/TAsset.hpp"
#include "GQE/GQE_types.hpp"
#include <SFML/Audio.hpp>

namespace GQE
{
  class MusicAsset : public TAsset<sf::Music>
  {
  public:
    /**
     * MusicAsset constructor
     * @param[in] theFilename to use when loading this asset
     * @param[in] theStyle to use when loading this asset
     */
    MusicAsset(std::string theFilename, AssetLoadingStyle theStyle = AssetLoadStyleBackground);

    /**
     * MusicAsset deconstructor
     */
    virtual ~MusicAsset();

    /**
     * LoadAsset is responsible for loading the Asset.
     */
    virtual void LoadAsset(void);

  protected:
    /**
     * UnloadAsset is responsible for destroying or unloading the Asset and
     * is called by FreeAsset.
     */
    virtual void UnloadAsset(void);

  private:
    // Variables
    ///////////////////////////////////////////////////////////////////////////
  }; // class MusicAsset
}; // namespace GQE

#endif // MUSIC_ASSET_HPP_INCLUDED
```

## <a name="25" />MusicAsset.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the Music Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/MusicAsset.cpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */

#include <assert.h>
#include <stddef.h>
#include "GQE/assets/MusicAsset.hpp"
#include "GQE/classes/App.hpp"

namespace GQE
{
  MusicAsset::MusicAsset(std::string theFilename, AssetLoadingStyle theStyle) :
    TAsset<sf::Music>(theFilename, theStyle)
  {
  }

  MusicAsset::~MusicAsset()
  {
    UnloadAsset();
  }

  void MusicAsset::LoadAsset(void)
  {
    // Only load the asset once if possible!
    if(false == mLoaded)
    {
      // Make sure memory is not already allocated
      assert(NULL == mAsset && "MusicAsset::LoadAsset() memory already allocated!");
    
      // Create the asset
      mAsset = new(std::nothrow) sf::Music;
      assert(NULL != mAsset && "MusicAsset::LoadAsset() unable to allocate memory");

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "MusicAsset::LoadAsset() loading asset with filename=" << mFilename << std::endl;
      }

      // Attempt to load the asset from a file
      mLoaded = mAsset->OpenFromFile(mFilename);

      // If the asset did not load successfully, delete the memory
      if(false == mLoaded)
      {
        // Output to log file
        if(NULL != mApp)
        {
          mApp->mLog << "MusicAsset::LoadAsset() asset with filename=" << mFilename << " is missing" << std::endl;
          mApp->Quit(StatusAppMissingAsset);
        }
      }
    }
  }

  void MusicAsset::UnloadAsset(void)
  {
    // If the music asset is currently loaded, make sure it is stopped
    if(true == mLoaded && NULL != mAsset)
    {
       // Always stop music from playing before removing from memory
       mAsset->Stop();
    }

    // Delete the asset, forcing it to be removed from memory
    delete mAsset;
    mAsset = NULL;
    mLoaded = false;
  }

}; // namespace GQE
```

## <a name="26" />SoundAsset.hpp [ [Top] ](#top)
```cpp
/**
 * Provides the Sound Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/SoundAsset.hpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */
#ifndef   SOUND_ASSET_HPP_INCLUDED
#define   SOUND_ASSET_HPP_INCLUDED

#include "GQE/interfaces/TAsset.hpp"
#include "GQE/GQE_types.hpp"
#include <SFML/Audio.hpp>

namespace GQE
{
  class SoundAsset : public TAsset<sf::SoundBuffer>
  {
  public:
    /**
     * SoundAsset constructor
     * @param[in] theFilename to use when loading this asset
     * @param[in] theStyle to use when loading this asset
     */
    SoundAsset(std::string theFilename, AssetLoadingStyle theStyle = AssetLoadStyleBackground);

    /**
     * SoundAsset deconstructor
     */
    virtual ~SoundAsset();

    /**
     * LoadAsset is responsible for loading the Asset.
     */
    virtual void LoadAsset(void);

  protected:
    /**
     * UnloadAsset is responsible for destroying or unloading the Asset and
     * is called by FreeAsset.
     */
    virtual void UnloadAsset(void);

  private:
    // Variables
    ///////////////////////////////////////////////////////////////////////////
  }; // class SoundAsset
}; // namespace GQE

#endif // SOUND_ASSET_HPP_INCLUDED
```

## <a name="27" />SoundAsset.cpp [ [Top] ](#top)
```cpp
/**
 * Provides the Sound Asset type used by the AssetManager in the GQE namespace
 * which is responsible for providing the Asset management facilities for the
 * App base class used by all applications.
 *
 * @file GQE/assets/SoundAsset.cpp
 * @author Ryan Lindeman
 * @date 20100724 - Initial Release
 */

#include <assert.h>
#include <stddef.h>
#include "GQE/assets/SoundAsset.hpp"
#include "GQE/classes/App.hpp"

namespace GQE
{
  SoundAsset::SoundAsset(std::string theFilename, AssetLoadingStyle theStyle) :
    TAsset<sf::SoundBuffer>(theFilename, theStyle)
  {
  }

  SoundAsset::~SoundAsset()
  {
    UnloadAsset();
  }

  void SoundAsset::LoadAsset(void)
  {
    // Only load the asset once if possible!
    if(false == mLoaded)
    {
      // Make sure memory is not already allocated
      assert(NULL == mAsset && "SoundAsset::LoadAsset() memory already allocated!");
    
      // Create the asset
      mAsset = new(std::nothrow) sf::SoundBuffer;
      assert(NULL != mAsset && "SoundAsset::LoadAsset() unable to allocate memory");

      // Output to log file
      if(NULL != mApp)
      {
        mApp->mLog << "SoundAsset::LoadAsset() loading asset with filename=" << mFilename << std::endl;
      }

      // Attempt to load the asset from a file
      mLoaded = mAsset->LoadFromFile(mFilename);

      // If the asset did not load successfully, delete the memory
      if(false == mLoaded)
      {
        // Output to log file
        if(NULL != mApp)
        {
          mApp->mLog << "SoundAsset::LoadAsset() asset with filename=" << mFilename << " is missing" << std::endl;
          mApp->Quit(StatusAppMissingAsset);
        }
      }
    }
  }

  void SoundAsset::UnloadAsset(void)
  {
    // Delete the asset, forcing it to be removed from memory
    delete mAsset;
    mAsset = NULL;
    mLoaded = false;
  }

}; // namespace GQE
```

## <a name="28" />GQE.hpp [ [Top] ](#top)
```cpp
/**
 * Provides all GQE namespace includes and global variables.
 *
 * @file GQE/GQE.hpp
 * @author Ryan Lindeman
 * @date 20100710 - Initial Release
 * @date 20110118 - Added ConfigReader and StateManager includes.
 */
#ifndef   GQE_HPP_INCLUDED
#define   GQE_HPP_INCLUDED
 
// GQE includes
#include "GQE/assets/ConfigAsset.hpp"
#include "GQE/assets/FontAsset.hpp"
#include "GQE/assets/ImageAsset.hpp"
#include "GQE/assets/MusicAsset.hpp"
#include "GQE/assets/SoundAsset.hpp"
#include "GQE/classes/App.hpp"
#include "GQE/classes/AssetManager.hpp"
#include "GQE/classes/ConfigReader.hpp"
#include "GQE/classes/StateManager.hpp"
#include "GQE/interfaces/IState.hpp"
#include "GQE/interfaces/TAsset.hpp"
#include "GQE/states/MenuState.hpp"
#include "GQE/states/SplashState.hpp"
 
namespace GQE
{
  /// Global variable declarations
}; // namespace GQE
 
#endif // GQE_HPP_INCLUDED
```

## <a name="29" />GQE_types.hpp [ [Top] ](#top)
```cpp
/**
 * Provides all GQE namespace typedef's and forward class declarations.
 *
 * @file GQE/GQE_types.hpp
 * @author Ryan Lindeman
 * @date 20100710 - Initial Release
 * @date 20110118 - Add types used by ConfigReader.
 * @date 20110125 - Fix string compare issues
 */
#ifndef   GQE_TYPES_HPP_INCLUDED
#define   GQE_TYPES_HPP_INCLUDED
 
#include <map>
#include <string>

// The following defines help with OS/Compiler specific calls
#ifdef WIN32
#ifndef MINGW
#define STRICMP _stricmp
#else
#define STRICMP strcasecmp
#endif
#else
#define STRICMP strcasecmp
#endif

namespace GQE
{
  /// Declare State ID typedef which is used for identifying State objects
  typedef std::string typeStateID;
 
  /// Declare Asset ID typedef which is used for identifying Asset objects
  typedef std::string typeAssetID;
 
  /// Declare NameValue typedef which is used for config section maps
  typedef std::map<const std::string, const std::string> typeNameValue;

  /// Declare NameValueIter typedef which is used for name,value pair maps
  typedef std::map<const std::string, const std::string>::iterator typeNameValueIter;

  /// Enumeration of all Asset loading styles
  enum AssetLoadingStyle {
    AssetLoadStyleFirst      = 0,  ///< First Loading Style
    AssetLoadStyleBackground = 1,  ///< Background thread loading style
    AssetLoadStyleForeground = 2,  ///< Foreground thread loading style
    AssetLoadStyleImmediate  = 3,  ///< Immediate loading style
    AssetLoadStyleLast             ///< Last Loading Style
  };
 
  /// Status Enumeration for Status Return values
  enum StatusType {
    // Values from -99 to 99 are common Error and Good status responses
    StatusAppMissingAsset = -4, ///< Application failed due to missing asset file
    StatusAppStackEmpty   = -3,  ///< Application States stack is empty
    StatusAppInitFailed   = -2,  ///< Application initialization failed
    StatusError           = -1,  ///< General error status response
    StatusAppOK           =  0,  ///< Application quit without error
    StatusNoError         =  0,  ///< General no error status response
    StatusFalse           =  0,  ///< False status response
    StatusTrue            =  1,  ///< True status response
    StatusOK              =  1   ///< OK status response
 
    // Values from +-100 to +-199 are reserved for File status responses
  };
 
  // Forward declare GQE interfaces provided
  class IState;

  // Forward declare GQE classes provided
  class App;
  class AssetManager;
  class ConfigReader;
  class StateManager;

  // Forward declare GQE assets provided
  class ConfigAsset;
  class FontAsset;
  class ImageAsset;
  class MusicAsset;
  class SoundAsset;
 
  // Forward declare GQE states provided
  class MenuState;
  class SplashState;
 
}; // namespace GQE
#endif // GQE_TYPES_HPP_INCLUDED
```

## <a name="30" />stdint.h [ [Top] ](#top)
If you are using Visual Studio 2008 or lower, you will need this file.  It defines the uint32_t, int32_t, and other typedef's.  Gcc and other compilers provide this file (since it is part of the C/C++ standard).

```cpp
// ISO C9x  compliant stdint.h for Microsoft Visual Studio
// Based on ISO/IEC 9899:TC2 Committee draft (May 6, 2005) WG14/N1124 
// 
//  Copyright (c) 2006-2008 Alexander Chemeris
// 
// Redistribution and use in source and binary forms, with or without
// modification, are permitted provided that the following conditions are met:
// 
//   1. Redistributions of source code must retain the above copyright notice,
//      this list of conditions and the following disclaimer.
// 
//   2. Redistributions in binary form must reproduce the above copyright
//      notice, this list of conditions and the following disclaimer in the
//      documentation and/or other materials provided with the distribution.
// 
//   3. The name of the author may be used to endorse or promote products
//      derived from this software without specific prior written permission.
// 
// THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS OR IMPLIED
// WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
// MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
// EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
// SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
// PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
// OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, 
// WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
// OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
// ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
// 
///////////////////////////////////////////////////////////////////////////////

#ifndef _MSC_VER // [
#error "Use this header only with Microsoft Visual C++ compilers!"
#endif // _MSC_VER ]

#ifndef _MSC_STDINT_H_ // [
#define _MSC_STDINT_H_

#if _MSC_VER > 1000
#pragma once
#endif

#include <limits.h>

// For Visual Studio 6 in C++ mode and for many Visual Studio versions when
// compiling for ARM we should wrap <wchar.h> include with 'extern "C++" {}'
// or compiler give many errors like this:
//   error C2733: second C linkage of overloaded function 'wmemchr' not allowed
#ifdef __cplusplus
extern "C" {
#endif
#  include <wchar.h>
#ifdef __cplusplus
}
#endif

// Define _W64 macros to mark types changing their size, like intptr_t.
#ifndef _W64
#  if !defined(__midl) && (defined(_X86_) || defined(_M_IX86)) && _MSC_VER >= 1300
#     define _W64 __w64
#  else
#     define _W64
#  endif
#endif


// 7.18.1 Integer types

// 7.18.1.1 Exact-width integer types

// Visual Studio 6 and Embedded Visual C++ 4 doesn't
// realize that, e.g. char has the same size as __int8
// so we give up on __intX for them.
#if (_MSC_VER < 1300)
   typedef signed char       int8_t;
   typedef signed short      int16_t;
   typedef signed int        int32_t;
   typedef unsigned char     uint8_t;
   typedef unsigned short    uint16_t;
   typedef unsigned int      uint32_t;
#else
   typedef signed __int8     int8_t;
   typedef signed __int16    int16_t;
   typedef signed __int32    int32_t;
   typedef unsigned __int8   uint8_t;
   typedef unsigned __int16  uint16_t;
   typedef unsigned __int32  uint32_t;
#endif
typedef signed __int64       int64_t;
typedef unsigned __int64     uint64_t;


// 7.18.1.2 Minimum-width integer types
typedef int8_t    int_least8_t;
typedef int16_t   int_least16_t;
typedef int32_t   int_least32_t;
typedef int64_t   int_least64_t;
typedef uint8_t   uint_least8_t;
typedef uint16_t  uint_least16_t;
typedef uint32_t  uint_least32_t;
typedef uint64_t  uint_least64_t;

// 7.18.1.3 Fastest minimum-width integer types
typedef int8_t    int_fast8_t;
typedef int16_t   int_fast16_t;
typedef int32_t   int_fast32_t;
typedef int64_t   int_fast64_t;
typedef uint8_t   uint_fast8_t;
typedef uint16_t  uint_fast16_t;
typedef uint32_t  uint_fast32_t;
typedef uint64_t  uint_fast64_t;

// 7.18.1.4 Integer types capable of holding object pointers
#ifdef _WIN64 // [
   typedef signed __int64    intptr_t;
   typedef unsigned __int64  uintptr_t;
#else // _WIN64 ][
   typedef _W64 signed int   intptr_t;
   typedef _W64 unsigned int uintptr_t;
#endif // _WIN64 ]

// 7.18.1.5 Greatest-width integer types
typedef int64_t   intmax_t;
typedef uint64_t  uintmax_t;


// 7.18.2 Limits of specified-width integer types

#if !defined(__cplusplus) || defined(__STDC_LIMIT_MACROS) // [   See footnote 220 at page 257 and footnote 221 at page 259

// 7.18.2.1 Limits of exact-width integer types
#define INT8_MIN     ((int8_t)_I8_MIN)
#define INT8_MAX     _I8_MAX
#define INT16_MIN    ((int16_t)_I16_MIN)
#define INT16_MAX    _I16_MAX
#define INT32_MIN    ((int32_t)_I32_MIN)
#define INT32_MAX    _I32_MAX
#define INT64_MIN    ((int64_t)_I64_MIN)
#define INT64_MAX    _I64_MAX
#define UINT8_MAX    _UI8_MAX
#define UINT16_MAX   _UI16_MAX
#define UINT32_MAX   _UI32_MAX
#define UINT64_MAX   _UI64_MAX

// 7.18.2.2 Limits of minimum-width integer types
#define INT_LEAST8_MIN    INT8_MIN
#define INT_LEAST8_MAX    INT8_MAX
#define INT_LEAST16_MIN   INT16_MIN
#define INT_LEAST16_MAX   INT16_MAX
#define INT_LEAST32_MIN   INT32_MIN
#define INT_LEAST32_MAX   INT32_MAX
#define INT_LEAST64_MIN   INT64_MIN
#define INT_LEAST64_MAX   INT64_MAX
#define UINT_LEAST8_MAX   UINT8_MAX
#define UINT_LEAST16_MAX  UINT16_MAX
#define UINT_LEAST32_MAX  UINT32_MAX
#define UINT_LEAST64_MAX  UINT64_MAX

// 7.18.2.3 Limits of fastest minimum-width integer types
#define INT_FAST8_MIN    INT8_MIN
#define INT_FAST8_MAX    INT8_MAX
#define INT_FAST16_MIN   INT16_MIN
#define INT_FAST16_MAX   INT16_MAX
#define INT_FAST32_MIN   INT32_MIN
#define INT_FAST32_MAX   INT32_MAX
#define INT_FAST64_MIN   INT64_MIN
#define INT_FAST64_MAX   INT64_MAX
#define UINT_FAST8_MAX   UINT8_MAX
#define UINT_FAST16_MAX  UINT16_MAX
#define UINT_FAST32_MAX  UINT32_MAX
#define UINT_FAST64_MAX  UINT64_MAX

// 7.18.2.4 Limits of integer types capable of holding object pointers
#ifdef _WIN64 // [
#  define INTPTR_MIN   INT64_MIN
#  define INTPTR_MAX   INT64_MAX
#  define UINTPTR_MAX  UINT64_MAX
#else // _WIN64 ][
#  define INTPTR_MIN   INT32_MIN
#  define INTPTR_MAX   INT32_MAX
#  define UINTPTR_MAX  UINT32_MAX
#endif // _WIN64 ]

// 7.18.2.5 Limits of greatest-width integer types
#define INTMAX_MIN   INT64_MIN
#define INTMAX_MAX   INT64_MAX
#define UINTMAX_MAX  UINT64_MAX

// 7.18.3 Limits of other integer types

#ifdef _WIN64 // [
#  define PTRDIFF_MIN  _I64_MIN
#  define PTRDIFF_MAX  _I64_MAX
#else  // _WIN64 ][
#  define PTRDIFF_MIN  _I32_MIN
#  define PTRDIFF_MAX  _I32_MAX
#endif  // _WIN64 ]

#define SIG_ATOMIC_MIN  INT_MIN
#define SIG_ATOMIC_MAX  INT_MAX

#ifndef SIZE_MAX // [
#  ifdef _WIN64 // [
#     define SIZE_MAX  _UI64_MAX
#  else // _WIN64 ][
#     define SIZE_MAX  _UI32_MAX
#  endif // _WIN64 ]
#endif // SIZE_MAX ]

// WCHAR_MIN and WCHAR_MAX are also defined in <wchar.h>
#ifndef WCHAR_MIN // [
#  define WCHAR_MIN  0
#endif  // WCHAR_MIN ]
#ifndef WCHAR_MAX // [
#  define WCHAR_MAX  _UI16_MAX
#endif  // WCHAR_MAX ]

#define WINT_MIN  0
#define WINT_MAX  _UI16_MAX

#endif // __STDC_LIMIT_MACROS ]


// 7.18.4 Limits of other integer types

#if !defined(__cplusplus) || defined(__STDC_CONSTANT_MACROS) // [   See footnote 224 at page 260

// 7.18.4.1 Macros for minimum-width integer constants

#define INT8_C(val)  val##i8
#define INT16_C(val) val##i16
#define INT32_C(val) val##i32
#define INT64_C(val) val##i64

#define UINT8_C(val)  val##ui8
#define UINT16_C(val) val##ui16
#define UINT32_C(val) val##ui32
#define UINT64_C(val) val##ui64

// 7.18.4.2 Macros for greatest-width integer constants
#define INTMAX_C   INT64_C
#define UINTMAX_C  UINT64_C

#endif // __STDC_CONSTANT_MACROS ]


#endif // _MSC_STDINT_H_ ]
```

## <a name="comments" />Comments [ [Top] ](#top)
Please leave your comments here or if you would appreciate a quick response, email my nick GatorQue in the forums.

Jmgr - You should put your getters const. To allow settings getters const and using a mutex, simply set your mutex as mutable by putting this keyword before his declaration.

eXpl0it3r - First of all a BIG thank you for the code and the tut! I don't know if you've ever run that script, but for me there're a lot of errors coming out of the compiler...
e.g. missing MenuState.hpp & SplashState.hpp or map error in ConfigReader.hpp and there seams to be many errors related to closing brackets and semicolons };
I'll try to fix somethings, but is there a fully working source? :-)

GatorQue - I have fixed all the compiler issues that I can find.  I also provided the MenuState and SplashState classes as examples of creating simple State classes.  I will add the CONST flags to the Getters/Setters in the near future.

eXpl0it3r - Thanks for the fix! It really works now! =)
Your code is for the SFML Version 1.6, I started playing around with v2.0 and had to make a few fixes for that, I ziped and uploaded it to this link:
http://www.rapidshare.com/files/443403165/GQE.zip Just in case someone would like that too. ;-)
Btw I've spotted a not relevant mistake, in MenuState.cpp and SplashState.cpp it's named in the comment section as .hpp instead of .cpp ;-) THX

GatorQue - I fixed the comments in the two State files and added the ability to add Inactive states to the StateManager class.  This allows you to add a bunch of states, whose Init will only be called when they become active.  The advantage to adding states as active is that their Init function will immediately be called, which might trigger the AssetManager to load certain images, sounds, etc.  If you want these assets to not be loaded, until the state becomes active, then add the state as an Inactive state.

jinho - Hi, wanted to say thanks so much for posting up your code, it's been really enlightening! This may be a stupid questions, but I was able to compile GQE, but when I try running it, it states that it can't load "MenuImage.png" and "SplashImage.png" - I put them in the same place as GQE.exe but they don't seem to do anything - I simply made the two images to be red and blue 1024 x 768 pngs, but they don't seem to load. What am I doing wrong? Where should the game assets be placed? Thanks!

GatorQue - The issue with the assets not loading is probably because your using Visual Studio to launch the program right after compiling.  When you launch the program from within Visual Studio it will usually use the "solution folder" as the working directory.  If you edit the properties of your project you can change the "Working Folder" under the "Debugging" category to "$(OutDir)" to force Visual Studio to use the folder of the executable as the working directory.  You could also put the assets in the same directory as your "solution file" too.

Mars_999 - Where can one download the whole project as a single .zip file for version 1.6? I see eXpl0it3r made a 2.0 project... Thanks!

GatorQue - Your in luck!  The GQE project can be downloaded from the GQE Google Code project here http://code.google.com/p/gqe/.  I will try and keep both the code above and the GQE project about the same, but eventually the GQE Google Code project will outpace the code above.