## Introduction
This guide provides comprehensive steps to install & configure the environment to cross-compile SFML for Android. If you have any issues, please post in this page's thread located at the SFML wiki forums.

## Download Dependencies
Before you can start, you'll need to download the following software.

* MinGW-64 - [here](https://developer.android.com/studio/index.html#downloads)
* Android Studio - [here](https://developer.android.com/studio/index.html#win-bundle)
* Android NDK **v12b** - [here](https://developer.android.com/ndk/downloads/older_releases.html)
* Git **Portable** - [here](https://git-scm.com/download/win)
* CMake Installer - [here](http://ant.apache.org/bindownload.cgi)
* Apache Ant - [here](http://ant.apache.org/bindownload.cgi)
* JDK Installer - [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

## Install Dependencies
* Install MinGW-64 to "c:\android\temp-mingw", move and rename "c:\android\temp-mingw\mingw32" to "c:\mingw".
* Install Android Studio to "c:\android\Android Studio" & "c:\android\sdk".
* Extract Android NDK into "c:\android\", rename directory to "ndk".
* Install Git Portable to "c:\android\git".
* Install CMake to "c:\android\cmake".
* Extract Apache Ant into "c:\android\", rename directory to "apache-ant".
* Install Java JDK.

## Configure Environment
* Open "c:\android\sdk\SDK Manager.exe", tick "Android 5.0.1 (API 21)", remember the API version you select.
* Open CMD as administrator from the start menu, paste the following command:
```
setx /M PATH "%PATH%;c:\android\cmake\bin;c:\android\git\bin;c:\android\sdk\tools;c:\android\sdk\platform-tools;c:\android\ndk;c:\android\apache-ant\bin;C:\mingw\bin" && setx /M ANDROID_NDK "c:/android/ndk"
```

## Compile SFML
*Open CMD.
*Run the following commands:
```
cd c:\android
git clone https://github.com/SFML/SFML.git SFML
cd SFML && mkdir build && cd build
mkdir armeabi-v7a && cd armeabi-v7a
set PATH=%PATH:c:\android\git\bin;=%
cmake -G "MinGW Makefiles" -DANDROID_ABI=armeabi-v7a -DCMAKE_TOOLCHAIN_FILE=../../cmake/toolchains/android.toolchain.cmake ../..
mingw32-make install
```

## Compile Android Example
*Plug your phone into your computer, enable USB debugging. Google how to if you do not know.
*Make sure you answer any prompts at any stage of running these next commands.
*Run the following commands:
```
cd c:\android\SFML\examples\android && rmdir bin /s /q
android update project --target "android-21" --path .
ndk-build
ant debug install && adb install bin/NativeActivity-debug.apk
adb shell am start -a android.intent.action.MAIN -n com.example.sfml/android.app.NativeActivity
```

## Bonus - Pong for Android
*I've tweaked the bundled Pong game to work slightly better on Android.
*Replace the contents of "c:\android\SFML\examples\android\jni\main.cpp" with the following:
```
#include <SFML/System.hpp>
#include <SFML/Window.hpp>
#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <SFML/Network.hpp>
#include <cmath>
#include <ctime>
#include <cstdlib>

// Do we want to showcase direct JNI/NDK interaction?
// Undefine this to get real cross-platform code.
#define USE_JNI

#if defined(USE_JNI)
// These headers are only needed for direct NDK/JDK interaction
#include <jni.h>
#include <android/native_activity.h>

// Since we want to get the native activity from SFML, we'll have to use an
// extra header here:
#include <SFML/System/NativeActivity.hpp>

// NDK/JNI sub example - call Java code from native code
int vibrate(sf::Time duration)
{
    // First we'll need the native activity handle
    ANativeActivity *activity = sf::getNativeActivity();
    
    // Retrieve the JVM and JNI environment
    JavaVM* vm = activity->vm;
    JNIEnv* env = activity->env;

    // First, attach this thread to the main thread
    JavaVMAttachArgs attachargs;
    attachargs.version = JNI_VERSION_1_6;
    attachargs.name = "NativeThread";
    attachargs.group = NULL;
    jint res = vm->AttachCurrentThread(&env, &attachargs);

    if (res == JNI_ERR)
        return EXIT_FAILURE;

    // Retrieve class information
    jclass natact = env->FindClass("android/app/NativeActivity");
    jclass context = env->FindClass("android/content/Context");
    
    // Get the value of a constant
    jfieldID fid = env->GetStaticFieldID(context, "VIBRATOR_SERVICE", "Ljava/lang/String;");
    jobject svcstr = env->GetStaticObjectField(context, fid);
    
    // Get the method 'getSystemService' and call it
    jmethodID getss = env->GetMethodID(natact, "getSystemService", "(Ljava/lang/String;)Ljava/lang/Object;");
    jobject vib_obj = env->CallObjectMethod(activity->clazz, getss, svcstr);
    
    // Get the object's class and retrieve the member name
    jclass vib_cls = env->GetObjectClass(vib_obj);
    jmethodID vibrate = env->GetMethodID(vib_cls, "vibrate", "(J)V"); 
    
    // Determine the timeframe
    jlong length = duration.asMilliseconds();
    
    // Bzzz!
    env->CallVoidMethod(vib_obj, vibrate, length);

    // Free references
    env->DeleteLocalRef(vib_obj);
    env->DeleteLocalRef(vib_cls);
    env->DeleteLocalRef(svcstr);
    env->DeleteLocalRef(context);
    env->DeleteLocalRef(natact);
    
    // Detach thread again
    vm->DetachCurrentThread();
}
#endif

// This is the actual Android example. You don't have to write any platform
// specific code, unless you want to use things not directly exposed.
// ('vibrate()' in this example; undefine 'USE_JNI' above to disable it)
int main(int argc, char *argv[])
{
    std::srand(static_cast<unsigned int>(std::time(NULL)));

    sf::RenderWindow window(sf::VideoMode::getDesktopMode(), "");
    window.setVerticalSyncEnabled(true);
    
    // Define some constants
    const float pi = 3.14159f;
    sf::Vector2u windowSizes = window.getSize();
    const int gameWidth = windowSizes.x;
    const int gameHeight = windowSizes.y;
    sf::Vector2f paddleSize(25, 300);
    float ballRadius = 15.f;

    // Load the sounds used in the game
    sf::SoundBuffer ballSoundBuffer;
    
    if (!ballSoundBuffer.loadFromFile("ball.wav"))
        return EXIT_FAILURE;
    sf::Sound ballSound(ballSoundBuffer);

    // Create the left paddle
    sf::RectangleShape leftPaddle;
    leftPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
    leftPaddle.setOutlineThickness(3);
    leftPaddle.setOutlineColor(sf::Color::Black);
    leftPaddle.setFillColor(sf::Color(100, 100, 200));
    leftPaddle.setOrigin(paddleSize / 2.f);

    // Create the right paddle
    sf::RectangleShape rightPaddle;
    rightPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
    rightPaddle.setOutlineThickness(3);
    rightPaddle.setOutlineColor(sf::Color::Black);
    rightPaddle.setFillColor(sf::Color(200, 100, 100));
    rightPaddle.setOrigin(paddleSize / 2.f);

    // Create the ball
    sf::CircleShape ball;
    ball.setRadius(ballRadius - 3);
    ball.setOutlineThickness(3);
    ball.setOutlineColor(sf::Color::Black);
    ball.setFillColor(sf::Color::White);
    ball.setOrigin(ballRadius / 2, ballRadius / 2);

    // Load the text font
    sf::Font font;
    if (!font.loadFromFile("sansation.ttf"))
        return EXIT_FAILURE;

    // Initialize the pause message
    sf::Text pauseMessage;
    pauseMessage.setFont(font);
    pauseMessage.setCharacterSize(40);
    pauseMessage.setPosition(170.f, 150.f);
    pauseMessage.setFillColor(sf::Color::White);
    pauseMessage.setString("Welcome to SFML pong!\Tap to start the game");

    // Define the paddles properties
    sf::Clock AITimer;
    const sf::Time AITime   = sf::seconds(0.1f);
    const float paddleSpeed = 600.f;
    float rightPaddleSpeed  = 0.f;
    const float ballSpeed   = 800.f;
    float ballAngle         = 0.f; // to be changed later

    sf::Clock clock;
    bool isPlaying = false;
    while (window.isOpen())
    {
        // Handle events
        sf::Event event;
        while (window.pollEvent(event))
        {
            // Window closed or escape key pressed: exit
            if ((event.type == sf::Event::Closed) ||
               ((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Escape)))
            {
                window.close();
                break;
            }

            // Tap to play
            if ((event.type == sf::Event::TouchBegan) && (event.touch.finger == 0))
            {
                if (!isPlaying)
                {
                    // (re)start the game
                    isPlaying = true;
                    clock.restart();

                    // Reset the position of the paddles and ball
                    leftPaddle.setPosition(100 + paddleSize.x / 2, gameHeight / 2);
                    rightPaddle.setPosition(gameWidth - 100 - paddleSize.x / 2, gameHeight / 2);
                    ball.setPosition(gameWidth / 2, gameHeight / 2);

                    // Reset the ball angle
                    do
                    {
                        // Make sure the ball initial angle is not too much vertical
                        ballAngle = (std::rand() % 360) * 2 * pi / 360;
                    }
                    while (std::abs(std::cos(ballAngle)) < 0.7f);
                }
            }
            
            if ((event.type == sf::Event::TouchMoved) && (event.touch.finger == 0))
            {
                sf::Vector2f oldPos = leftPaddle.getPosition();
                leftPaddle.setPosition(oldPos.x, event.touch.y);
            }
        }

        if (isPlaying)
        {
            float deltaTime = clock.restart().asSeconds();
            /*sf::Vector2f oldPos = object.getPosition();

            // Move the player's paddle
            if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) &&
               (leftPaddle.getPosition().y - paddleSize.y / 2 > 5.f))
            {
                //leftPaddle.move(0.f, -paddleSpeed * deltaTime);
                leftPaddle.setPosition(oldPos.x, event.touch.y);
            }
            if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down) &&
               (leftPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f))
            {
                //leftPaddle.move(0.f, paddleSpeed * deltaTime);
                leftPaddle.setPosition(oldPos.x, event.touch.y);
            }*/

            // Move the computer's paddle
            if (((rightPaddleSpeed < 0.f) && (rightPaddle.getPosition().y - paddleSize.y / 2 > 5.f)) ||
                ((rightPaddleSpeed > 0.f) && (rightPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f)))
            {
                rightPaddle.move(0.f, rightPaddleSpeed * deltaTime);
            }

            // Update the computer's paddle direction according to the ball position
            if (AITimer.getElapsedTime() > AITime)
            {
                AITimer.restart();
                if (ball.getPosition().y + ballRadius > rightPaddle.getPosition().y + paddleSize.y / 2)
                    rightPaddleSpeed = paddleSpeed;
                else if (ball.getPosition().y - ballRadius < rightPaddle.getPosition().y - paddleSize.y / 2)
                    rightPaddleSpeed = -paddleSpeed;
                else
                    rightPaddleSpeed = 0.f;
            }

            // Move the ball
            float factor = ballSpeed * deltaTime;
            ball.move(std::cos(ballAngle) * factor, std::sin(ballAngle) * factor);

            // Check collisions between the ball and the screen
            if (ball.getPosition().x - ballRadius < 0.f)
            {
                isPlaying = false;
                pauseMessage.setString("You lost!\nTap to restart or\nescape to exit");
            }
            if (ball.getPosition().x + ballRadius > gameWidth)
            {
                isPlaying = false;
                pauseMessage.setString("You won!\nTap to restart or\nescape to exit");
            }
            if (ball.getPosition().y - ballRadius < 0.f)
            {
                ballSound.play();
                ballAngle = -ballAngle;
                ball.setPosition(ball.getPosition().x, ballRadius + 0.1f);
            }
            if (ball.getPosition().y + ballRadius > gameHeight)
            {
                ballSound.play();
                ballAngle = -ballAngle;
                ball.setPosition(ball.getPosition().x, gameHeight - ballRadius - 0.1f);
            }

            // Check the collisions between the ball and the paddles
            // Left Paddle
            if (ball.getPosition().x - ballRadius < leftPaddle.getPosition().x + paddleSize.x / 2 &&
                ball.getPosition().x - ballRadius > leftPaddle.getPosition().x &&
                ball.getPosition().y + ballRadius >= leftPaddle.getPosition().y - paddleSize.y / 2 &&
                ball.getPosition().y - ballRadius <= leftPaddle.getPosition().y + paddleSize.y / 2)
            {
                if (ball.getPosition().y > leftPaddle.getPosition().y)
                    ballAngle = pi - ballAngle + (std::rand() % 20) * pi / 180;
                else
                    ballAngle = pi - ballAngle - (std::rand() % 20) * pi / 180;

                ballSound.play();
                ball.setPosition(leftPaddle.getPosition().x + ballRadius + paddleSize.x / 2 + 0.1f, ball.getPosition().y);
                
                #if defined(USE_JNI)
                        vibrate(sf::milliseconds(10));
                #endif
            }

            // Right Paddle
            if (ball.getPosition().x + ballRadius > rightPaddle.getPosition().x - paddleSize.x / 2 &&
                ball.getPosition().x + ballRadius < rightPaddle.getPosition().x &&
                ball.getPosition().y + ballRadius >= rightPaddle.getPosition().y - paddleSize.y / 2 &&
                ball.getPosition().y - ballRadius <= rightPaddle.getPosition().y + paddleSize.y / 2)
            {
                if (ball.getPosition().y > rightPaddle.getPosition().y)
                    ballAngle = pi - ballAngle + (std::rand() % 20) * pi / 180;
                else
                    ballAngle = pi - ballAngle - (std::rand() % 20) * pi / 180;

                ballSound.play();
                ball.setPosition(rightPaddle.getPosition().x - ballRadius - paddleSize.x / 2 - 0.1f, ball.getPosition().y);
            }
        }

        // Clear the window
        window.clear(sf::Color(50, 200, 50));

        if (isPlaying)
        {
            // Draw the paddles and the ball
            window.draw(leftPaddle);
            window.draw(rightPaddle);
            window.draw(ball);
        }
        else
        {
            // Draw the pause message
            window.draw(pauseMessage);
        }

        // Display things on screen
        window.display();
    }
}
```