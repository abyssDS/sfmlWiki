This wiki page contains the class FPS which implements the following two functions for calculation the framerate once per second:

* FPS::update() - Must be called at the end of every frame. This calculates and resets values if needed.
* FPS::getFPS() - Returns the current framerate as an `const unsigned int`.

### Code - FPS.hpp
```cpp
#ifndef FPS_HPP
#define FPS_HPP

class FPS
{ 
public:
	/// @brief Constructor with initialization.
	///
	FPS() : mFrame(0), mFps(0) {}

	/// @brief Update the frame count.
	/// 
	void update();

	/// @brief Get the current FPS count.
	/// @return FPS count.
	const unsigned int getFPS() const { return mFps; }

private:
	unsigned int mFrame;
	unsigned int mFps;
	sf::Clock mClock;
};

void FPS::update()
{
	if(mClock.getElapsedTime().asSeconds() >= 1.f)
	{
		mFps = mFrame;
		mFrame = 0;
		mClock.restart();
	}
 
	++mFrame;
}

#endif // FPS_HPP
```
-----------
### Another example of an implementation that calculates the FPS on a per frame basis:
```cpp
float getFPS(const sf::Time& time) {
     return (1000000.0f / time.asMicroseconds());
}

sf::Clock FPSClock;
while(rungame) {
    /*

    Game logic goes here   

    */
    float fps = getFPS(FPSClock.restart());
}
```