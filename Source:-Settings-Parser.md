#SFML

## Overview
This is simple class for reading and writing settings in text format. It allow to store std::string, bool, char, int and float.

### Simple settings file

```text
# if line starts with # this line will be ignored
# black lines also

# screen size
width = 1024
height = 768

# windows title
title = sfml tutorial

# phycics constants
g = 9.81

# player initials
player = x

# video mode
use30hz = TRUE

```

### Simple using example

```cpp
int main()
{
	//....
	
	SettingsParser settings("settings.dat");
	
	int width, height;
	settings.Get("width", &width);
	settings.Get("height", &height);
	
	std::string str;
	settings.Get("title", &str);
	
	float gravity;
	settings.Get("g", &gravity);
	
	char ch;
	settings.Get("player", &ch);
	
	bool isLow;
	settings.Get("use30hz", &isLow);
	
	//....
	
	settings.Set("title", "sfml tutorial");
	settings.Write();
	return 0;
}
```


## Source code

### SettingsParser.h
```cpp
#ifndef SETTINGS_H
#define SETTINGS_H

#include <fstream>
#include <iostream>
#include <string>
#include <vector>
#include <sstream>
#include <utility>

class Settings
{
protected:
	bool isChanged;
	std :: string filename;
	std :: vector < std :: pair < std :: string, std :: string> > data;
	size_t size, max_width;
public:
	Settings(std :: string file);
	Settings(std :: string file, size_t width);
	~Settings();
		
	void Read();
	void Write();

	bool IsChanged();

	void Get(std :: string param, std :: string* value);
	void Get(std :: string param, bool* value);
	void Get(std :: string param, char* value);
	void Get(std :: string param, int* value);
	void Get(std :: string param, float* value);

	void Set(std :: string param, std :: string value);
	void Set(std :: string param, bool value);
	void Set(std :: string param, char value);
	void Set(std :: string param, int value);
	void Set(std :: string param, float value);
};

#endif // SETTINGS_H
```

### SettingsParser.cpp
```cpp
#include "Settings.h"

Settings :: Settings(std :: string file)
{
	filename = file;
	max_width = 100;
	size = 0;
	Read();
	isChanged = false;
}

Settings :: Settings(std :: string file, size_t width)
{
	filename = file;
	max_width = width;
	size = 0;
	Read();
}

Settings :: ~Settings()
{
	if(isChanged)
		Write();
	filename.clear();
	data.clear();
}

void Settings :: Read()
{
	std :: ifstream in(filename.c_str());
	if(!in.is_open())
	{
		std :: cerr << "error at reading" << std :: endl;
		return;
	}
	std :: string line, param, value;
	while(!in.eof())
	{
		std::getline(in, line);
		if(line.size() > 0 && line[0] != '#')
		{
			size_t j = 0;
			size_t length = line.size();
			while(line[j] != ' ') j++;
			param = line.substr(0,j);
			while(line[j] == ' ' || line[j] == '=') j++;
			int a = j;
			while(j < length && (line[j] != ' ' || line[j] != '\n')) j++;
			int b = j;
			value = line.substr(a, b);
		}
		else
		{
			param = line;
			value = "";
		}
		data.push_back(make_pair(param, value));
		size++;
	}
	in.close();
	isChanged = true;
}

void Settings :: Write()
{
	std :: ofstream out(filename.c_str());
	if(!out.is_open())
	{
		std :: cerr << "error at writing" << std :: endl;
		return;
	}
	for(size_t i = 0; i < size - 1; ++i)
	{
		if(data[i].first[0] == '#' || data[i].first[0] == 0)
			out << data[i].first << std :: endl;
		else
			out << data[i].first << " = " << data[i].second << std :: endl;
	}
	out.close();
}

bool Settings :: IsChanged()
{
	return isChanged;
}

void Settings :: Get(std :: string param, std :: string* value)
{
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			*value = data[i].second;
			return;
		}
	}
}

void Settings :: Get(std :: string param, bool* value)
{
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			*value = ((data[i].second == "TRUE") ? true : false);
			return;
		}
	}
}

void Settings :: Get(std :: string param, char* value)
{
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			*value = data[i].second[0];
			return;
		}
	}
}

void Settings :: Get(std :: string param, int* value)
{
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			std::stringstream ss(data[i].second);
			ss >> *value;
			return;
		}
	}
}

void Settings :: Set(std :: string param, std :: string value)
{
	isChanged = true;
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			data[i].second = value;
			return;
		}
	}
}

void Settings :: Set(std :: string param, bool value)
{
	isChanged = true;
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			data[i].second = (value) ? "TRUE" : "FALSE";
			return;
		}
	}
}

void Settings :: Set(std :: string param, char value)
{
	isChanged = true;
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			std::string tmp = "";
			tmp = value;
			data[i].second = tmp;
			return;
		}
	}
}

void Settings :: Set(std :: string param, int value)
{
	isChanged = true;
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			std::stringstream ss;
			ss << value;
			data[i].second = ss.str();
			return;
		}
	}
}

void Settings :: Set(std :: string param, float value)
{
	isChanged = true;
	for(size_t i = 0; i < size; ++i)
	{
		if(data[i].first[0] == '#')
			continue;
		if(data[i].first == param)
		{
			std::stringstream ss;
			ss << value;
			data[i].second = ss.str();
			return;
		}
	}
}
```