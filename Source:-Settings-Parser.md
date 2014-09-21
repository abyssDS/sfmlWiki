#SFML

## Overview
This is simple class for reading and writing settings in a human readable from a text file. It allows you to read a `std::string`, `bool`, `char`, `int` or `float` from a file. You can then access these values from within your program. You may also change the values and write them back to the file. The syntax is basically a `key = value` pair.

Please note that the keys have to be unique. If the same name is used twice, the value will be overwritten.

The original author is cristaloleg, but the class was completely refractored and improved by [Foaly](https://github.com/Foaly). 
It can also be found in [this repository](https://github.com/Foaly/SettingsParser).

### Simple settings file

```text
# if a line starts with a '#' the line is a comment and will be ignored
# blank lines will also be ignored

# the syntax is
key = value

# screen size
width = 1024
height = 768

# window title
title = sfml tutorial

# phycics constants
g = 9.81

# player initials (since the key is the same the value will be overwritten)
player = M
player = X

# video mode
fullscreen = TRUE

```

### Simple usage example

```cpp
#include "SettingsParser.hpp"
#include <iostream>

int main()
{
    //....

    SettingsParser settings;
    if(!settings.loadFromFile("settings.txt"))
    {
        std::cout << "Error loading settings file!" << std::endl;
        return -1;
    }

    int width = 800;
    int height = 600;
    settings.get("width", width);
    settings.get("height", height);
    // if "width" and "height" are not found in the settings file their values are untouched 
    // (meaning you can easily set defaults like shown above)

    std::string title;
    settings.get("title", title);

    float gravity;
    settings.get("g", gravity);

    char ch;
    settings.get("player", ch);

    bool fullscreen;
    settings.get("fullscreen", fullscreen);

    //....

    std::string newTitle = "SFML Settings Parser rocks!";
    settings.set("title", "SFML Settings Parser rocks!");
    settings.saveToFile(); // this is also done in the destructor

    settings.print();
    std::cin.get();

    return 0;
}
```


## Source code

### SettingsParser.hpp
```cpp
#ifndef SETTINGSPARSER_INCLUDE
#define SETTINGSPARSER_INCLUDE

#include <string>
#include <map>

class SettingsParser 
{
public:
    SettingsParser();
    ~SettingsParser();

    bool loadFromFile(const std::string& filename);
    bool saveToFile();

    bool isChanged() const;

    void get(const std::string& key, std::string &value) const;
    void get(const std::string& key, bool &value) const;
    void get(const std::string& key, char &value) const;
    void get(const std::string& key, int &value) const;
    void get(const std::string& key, float &value) const;

    void set(const std::string& key, const std::string value);
    void set(const std::string& key, const char* value);
    void set(const std::string& key, const bool value);
    void set(const std::string& key, const char value);
    void set(const std::string& key, const int value);
    void set(const std::string& key, const float value);

    void print() const;


private:
    bool read();
    bool write() const;

    bool m_isChanged;
    std::string m_filename;
    std::map<std::string, std::string> m_data;
};

#endif // SETTINGSPARSER_INCLUDE
```

### SettingsParser.cpp
```cpp
#include "SettingsParser.hpp"

#include <locale>
#include <fstream>
#include <iostream>
#include <sstream>
#include <vector>


SettingsParser::SettingsParser() : m_isChanged(false)
{

}


SettingsParser::~SettingsParser()
{
    saveToFile();
}


bool SettingsParser::loadFromFile(const std::string& filename)
{
    m_data.clear();
    m_filename = filename;
    return read();
}


bool SettingsParser::saveToFile()
{
    if(m_isChanged)
    {
        m_isChanged = false;
        return write();
    }
    return true;
}


bool SettingsParser::read()
{
    std::ifstream in(m_filename);
    if(!in.is_open())
    {
        std::cerr << "Error: Unable to open settings file: \"" << m_filename << "\" for reading!" << std::endl;
        return false;
    }
    std::string line;
    const std::locale loc;
    while(std::getline(in, line))
    {
        // parse line
        if(line.size() > 0 && line[0] != '#')
        {
            size_t j = 0;
            // trim leading whitespace
            while(std::isspace(line[j], loc))
                j++;
            // get the key string
            const int beginKeyString = j;
            while(!std::isspace(line[j], loc))
                j++;
            const std::string key = line.substr(beginKeyString, j - beginKeyString);

            // skip the assignment
            while(std::isspace(line[j], loc) || line[j] == '=')
                j++;
            
            // get the value string
            const std::string value = line.substr(j, line.size() - j);

            m_data[key] = value;
        }
    }
    in.close();
    m_isChanged = false;
    return true;
}


bool SettingsParser::write() const
{
    std::vector<std::pair<std::string, std::string> > fileContents;

    std::ifstream in(m_filename);
    if(in.is_open())
    {
        std::string line, key, value;
        const std::locale loc;
        while(std::getline(in, line))
        {
            // parse line
            if(line.size() > 0 && line[0] != '#')
            {
                size_t j = 0;
                // trim leading whitespace
                while(std::isspace(line[j], loc))
                    j++;
                // get the key string
                const int beginKeyString = j;
                while(!std::isspace(line[j], loc))
                    j++;
                key = line.substr(beginKeyString, j - beginKeyString);

                // check if the key is found in the map
                std::map<std::string, std::string>::const_iterator it = m_data.find(key);
                if (it != m_data.end())
                {
                    // if so take it's value
                    value = it->second;
                }
                else
                {
                    // if not get the value string from the file

                    // skip the assignment
                    while(std::isspace(line[j], loc) || line[j] == '=')
                        j++;
                
                    // get the value string
                    value = line.substr(j, line.size() - j);
                }
            }
            else
            {
                key = line;
                value = "";
            }
            fileContents.push_back(std::make_pair(key, value));
        }
    }
    else
    {
        // Can't open file for reading. Use only the data from the map
        for (std::map<std::string, std::string>::const_iterator it = m_data.begin(); it != m_data.end(); ++it)
            fileContents.push_back(std::make_pair(it->first, it->second));
    }

    in.close();



    // open the file for writing
    std::ofstream out(m_filename);
    if(!out.is_open())
    {
        std::cerr << "Error: Unable to open settings file: \"" << m_filename << "\" for writing!" << std::endl;
        return false;
    }
    for (std::vector<std::pair<std::string, std::string> >::const_iterator it = fileContents.begin() ; it != fileContents.end(); ++it)
    {
        if(it->first[0] == '#' || it->first[0] == 0)
            out << it->first << std::endl;
        else
            out << it->first << " = " << it->second << std::endl;
    }
    out.close();
    return true;
}


void SettingsParser::print() const
{
    for(auto& element: m_data)
        std::cout << element.first << " = " << element.second<< std::endl;

    std::cout << std::endl << "Size: " << m_data.size() << std::endl;
}


bool SettingsParser::isChanged() const
{
    return m_isChanged;
}


void SettingsParser::get(const std::string& key, std::string &value) const
{
    std::map<std::string, std::string>::const_iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        value = it->second;
    }
}


void SettingsParser::get(const std::string& key, bool &value) const
{
    std::map<std::string, std::string>::const_iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        value = ((it->second == "TRUE") ? true : false);
    }
}


void SettingsParser::get(const std::string& key, char &value) const
{
    std::map<std::string, std::string>::const_iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        value = it->second[0];
    }
}


void SettingsParser::get(const std::string& key, int &value) const
{
    std::map<std::string, std::string>::const_iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        std::stringstream ss(it->second);
        ss >> value;
    }
}


void SettingsParser::get(const std::string& key, float &value) const
{
    std::map<std::string, std::string>::const_iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        std::stringstream ss(it->second);
        ss >> value;
    }
}


void SettingsParser::set(const std::string& key, const std::string value)
{
    std::map<std::string, std::string>::iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        it->second = value;
        m_isChanged =  true;
    }                                 
}

void SettingsParser::set(const std::string& key, const char* value)
{
    std::map<std::string, std::string>::iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        it->second = std::string(value);
        m_isChanged =  true;
    }                                 
}


void SettingsParser::set(const std::string& key, const bool value)
{
    std::map<std::string, std::string>::iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        it->second = (value) ? "TRUE" : "FALSE";
        m_isChanged =  true;
    }
}


void SettingsParser::set(const std::string& key, const char value)
{
    std::map<std::string, std::string>::iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        std::string tmp = "";
        tmp = value;
        it->second = tmp;
        m_isChanged =  true;
    }
}


void SettingsParser::set(const std::string& key, const int value)
{
    std::map<std::string, std::string>::iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        std::stringstream ss;
        ss << value;
        it->second = ss.str();
        m_isChanged =  true;
    }
}


void SettingsParser::set(const std::string& key, const float value)
{
    std::map<std::string, std::string>::iterator it = m_data.find(key);
    if (it != m_data.end())
    {
        std::stringstream ss;
        ss << value;
        it->second = ss.str();
        m_isChanged =  true;
    }
}
```