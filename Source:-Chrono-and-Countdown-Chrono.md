# Chrono and countdown chrono
	Simple chronometer and countdown chronometer, that you could
	include into your game engine, in order to handle timing.
	
![Spritesheet](https://i.imgur.com/nqIbWpP.png)

**Chrono.h**
```c++
/*
	Keyboard keys:
	<R>: RUN/RESUME
	<S>: STOP
	<P>: PAUSE
	<T>: RESET/RERUN

	The Chrono class code could be used by any other framework or lib, simply
	change the returned *(unsigned int get_ticks()) class's private member method implementation.
		exp.: with SDL libs. *(unsigned int get_ticks()) would implements *(Uint32 SDL_GetTicks()) API.

	Any code improvement will be very appreciated.
*/

#ifndef CHRONO_H
#define CHRONO_H

//
#include <SFML/System.hpp>
// or
// #include <SDL.h>
// or ... Any time handling  API 

class Chrono
{
private:
	unsigned int m_ticks;
	unsigned int m_catched_ticks;
	unsigned int m_enlapsed_ticks;
	unsigned int m_remaining_ticks;

	bool m_is_enable;
	bool m_is_stopped;
	bool m_is_to_be_resumed;

	unsigned int m_duration; // In case of Rebours account
	bool m_is_rebours;

	sf::Clock m_clock;
private:
	unsigned int get_ticks() const;

public:
	// Construct the object's initial state
	// If the duration arguments is set to 0, 
	// the count of time will be computed as non-rebours
	Chrono();
	Chrono(const bool& is_enabled);
	Chrono(const unsigned int& duration);
	Chrono(const unsigned int& duration, const bool& is_enabled);
	~Chrono();

	void Enable();
	void Disable();
	void Set_Duration(const unsigned int& duration);
	unsigned int Get_duration() const;

	void Reset_Rerun();
	void Stop();
	void Pause();
	void Run_Resume();

	unsigned int Get_Ticks();

	inline bool Is_Enable() const;
};
#endif // !CHRONO_H
```

**Chrono.cpp**

```c++
/*
	The Chrono class code could be used by any other framework or lib, simply
	change the returned *(unsigned int get_ticks()) class's private member method implementation.
		exp.: with SDL libs. *(unsigned int get_ticks()) would implements *(Uint32 SDL_GetTicks()) API.

	Any code improvement will be very appreciated.
	Yassir KENNY - tayssirdev@gmail.com
*/

#include "Chrono.h"

Chrono::Chrono():
	m_is_enable(false), m_is_stopped(true), m_is_to_be_resumed(false), m_is_rebours(false),
	m_ticks(0), m_catched_ticks(0), m_enlapsed_ticks(0),
	m_duration(0), m_remaining_ticks(0)
{

}

Chrono::Chrono(const bool & is_enabled):
	m_is_enable(is_enabled), m_is_stopped(true), m_is_to_be_resumed(false), m_is_rebours(false),
	m_ticks(0), m_catched_ticks(0), m_enlapsed_ticks(0),
	m_duration(0), m_remaining_ticks(0)
{
}

Chrono::Chrono(const unsigned int & duration) :
	m_is_enable(false), m_is_stopped(true), m_is_to_be_resumed(false), m_is_rebours(false),
	m_ticks(0), m_catched_ticks(0), m_enlapsed_ticks(0),
	m_duration(duration), m_remaining_ticks(0)
{
}

Chrono::Chrono(const unsigned int & duration, const bool & is_enabled):
	m_is_enable(is_enabled), m_is_stopped(true), m_is_to_be_resumed(false), m_is_rebours(false),
	m_ticks(0), m_catched_ticks(0), m_enlapsed_ticks(0),
	m_duration(duration), m_remaining_ticks(0)
{

}

Chrono::~Chrono()
{
	// No heap objects!, so let the system do the clean work instead.
}

void Chrono::Enable()
{
	m_is_enable = true;
}

void Chrono::Disable()
{
	m_is_enable = false;
}

void Chrono::Set_Duration(const unsigned int & duration)
{
// If the duration arguments is set to 0, 
// the count of time will be considered as non-rebours
	if (duration != 0)
	{
		m_duration = duration;
		m_is_rebours = true;
	}
	else
	{
		m_duration = duration;
		m_is_rebours = false;
	}
}

unsigned int Chrono::Get_duration() const
{
	return m_duration;
}

void Chrono::Reset_Rerun()
{
	m_is_to_be_resumed = false;
	m_is_stopped = false;

	if (m_is_rebours)
		m_remaining_ticks = m_duration;

	m_catched_ticks = get_ticks();
}

void Chrono::Stop()
{
	m_is_stopped = true;
	m_is_to_be_resumed = false;
}

void Chrono::Pause()
{
	m_is_stopped = false;
	m_is_to_be_resumed = true;
}

void Chrono::Run_Resume()
{
	m_is_stopped = false;
	m_is_to_be_resumed = false;

	m_catched_ticks = (get_ticks() - m_enlapsed_ticks);
}

unsigned int Chrono::Get_Ticks()
{
	if (m_is_enable) // Use chrono
	{
		if (m_is_stopped) // Stop
		{
			m_catched_ticks = m_enlapsed_ticks = m_ticks = 0;
			
			if (m_is_rebours)
			{
				m_remaining_ticks = 0;
				return m_remaining_ticks;
			}

			return m_ticks;
		}
		else if (m_is_to_be_resumed) // Pause
		{
			if (m_is_rebours)
				return m_remaining_ticks;
			
			return m_enlapsed_ticks;
		}
		else // Run-Resume, Reset-Rerun 
		{
			m_ticks = (get_ticks() - m_catched_ticks);
			m_enlapsed_ticks = m_ticks;
			if (m_is_rebours)
			{
				m_remaining_ticks = m_duration - m_enlapsed_ticks;
				if (m_enlapsed_ticks >= m_duration)
				{
					m_remaining_ticks = 0;
					m_is_stopped = true;
				}
				return m_remaining_ticks;
			}
			return m_ticks;
		}
	}
	else // Do not use chrono
	{
		return 0;
	}
}

inline bool Chrono::Is_Enable() const
{
	return m_is_enable;
}

// 
unsigned int Chrono::get_ticks() const
{
	return m_clock.getElapsedTime().asMilliseconds(); // SFML API
	//return static_cast<unsigned int>(SDL_GetTicks()); // SDL API
}
```
**main.cpp**
``` C++
/*
	Keyboard keys:
	<R>: RUN/RESUME
	<S>: STOP
	<P>: PAUSE
	<T>: RESET/RERUN

	The Chrono class code could be used by any other framework or lib, simply
	change the returned *(unsigned int get_ticks()) class's private member method implementation.
		exp.: with SDL libs. *(unsigned int get_ticks()) would implements *(Uint32 SDL_GetTicks()) API.

	Any code improvement will be very appreciated.

	Yassir KENNY - tayssirdev@gmail.com
*/

#include <SFML/System.hpp>
#include <SFML/System/String.hpp>
//
#include <SFML/Window/Keyboard.hpp>
#include <SFML/Window/Event.hpp>
//
#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/Graphics/Font.hpp>
#include <SFML/Graphics/Text.hpp>
#include <SFML/Graphics/Color.hpp>
//
#include <iostream>
#include <string>
#include <vector>
//
#include "Chrono.h"


int main(int argc, char *argv[])
{
	//	--------------------------------------------------
	Chrono _chrono;
	_chrono.Enable();
	_chrono.Set_Duration(2000); // 0: Normal count, !0: Rebours account
	unsigned int _ticks(0);
	//----------------------------------------------------
	
	
	std::vector<unsigned int> _dimensions = {640,360};
	std::string _title("Chrono");
	//
	sf::RenderWindow _wnd;
	sf::VideoMode _wnd_video_mode(_dimensions[0], _dimensions[1]);
	std::string _wnd_title = _title;
	//
	_wnd.create(_wnd_video_mode, static_cast<sf::String>(_wnd_title));
	
	/*
	   / \
	  / ! \	 In order to display things, don't forget to load a font_file.
	 /____ \ 
	 https://www.sfml-dev.org/tutorials/2.5/graphics-text.php
	*/
	sf::Font _font;
	if (!(_font.loadFromFile("font_file.ttf")))
		std::cout << "=[X]=: Loading font file." << std::endl;
	else
		std::cout << "=[i]=: SUCCESS loading font file." << std::endl;

	// --------------------------------------------------
	sf::Text _text_info;
	_text_info.setFont(_font);
	_text_info.setCharacterSize(15);
	_text_info.setFillColor(sf::Color::White);
	_text_info.setStyle(sf::Text::Regular);
	_text_info.setPosition(5, 10);


	std::string _tui("\n");
	_tui.append("<R>: RUN/RESUME \t");
	_tui.append("<S>: STOP \t");
	_tui.append("<P>: PAUSE \t");
	_tui.append("<T>: RESET/RERUN \t");
	_tui.append("\n");
	unsigned int  _v = _chrono.Get_duration();
	if (_v != 0)
	{
	_tui.append("Rebours account - Duration : ");
	_tui.append(std::to_string(_v) + " ms");
	}
	
	sf::Text _text_chrono;
	_text_chrono.setFont(_font);
	_text_chrono.setCharacterSize(72);
	_text_chrono.setFillColor(sf::Color::Green);
	_text_chrono.setStyle(sf::Text::Bold);
	_text_chrono.setPosition(200, 150);
	
	//
	sf::Event _event;
	while (_wnd.isOpen())
	{
		while (_wnd.pollEvent(_event))
		{
			//***** Windowing system keyboard handling *******
			if (_event.key.code == sf::Keyboard::Key::Escape)
				_wnd.close();
			
			//-------------------------------
			switch (_event.key.code)
			{
				case sf::Keyboard::Key::P:
					_chrono.Pause();
					break;
				case sf::Keyboard::Key::R: 
					_chrono.Run_Resume();
					break;
				case sf::Keyboard::Key::S:
					_chrono.Stop();
					break;
				case sf::Keyboard::Key::T:
					_chrono.Reset_Rerun();
					break;
			}
		}
		
		_ticks = _chrono.Get_Ticks();
		
		//***** Set Render *******
		_text_info.setString(_tui);
		_text_chrono.setString(std::to_string(_ticks));
		//*****   Render   *******
		_wnd.clear();
			_wnd.draw(_text_info);
			_wnd.draw(_text_chrono);
		_wnd.display();
	}
	return 0;
}
```