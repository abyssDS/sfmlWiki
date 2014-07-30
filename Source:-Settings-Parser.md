#SFML

## Overview
This is simple class for reading and writing settings in text format. It allows you to store std::string, bool, char, int and float in a file.

The original author is cristaloleg, but the class was completely refractored and improved by Foaly.

### Simple settings file

```text
# if a line starts with a '#' the line will be ignored
# blank lines will also be ignored

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

    SettingsParser settings;
    if(!settings.loadFile("settings.dat"))
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

    std::string str;
    settings.get("title", str);

    float gravity;
    settings.get("g", gravity);

    char ch;
    settings.get("player", ch);

    bool isLow;
    settings.get("use30hz", isLow);

    //....

    settings.set("title", "SFML Settings Parser rocks!");
    settings.saveToFile(); // this is also done in the destructor
    return 0;
}
```


## Source code

### SettingsParser.hpp
```cpp
#ifndef SETTINGSPARSER_INCLUDE
#define SETTINGSPARSER_INCLUDE

#include <fstream>
#include <iostream>
#include <string>
#include <vector>
#include <sstream>

class SettingsParser 
{
public:
    SettingsParser();
    ~SettingsParser();

    bool loadFile(const std::string& filename);
    bool saveToFile() const;

    bool isChanged() const;

    void get(const std::string& param, std::string &value) const;
    void get(const std::string& param, bool &value) const;
    void get(const std::string& param, char &value) const;
    void get(const std::string& param, int &value) const;
    void get(const std::string& param, float &value) const;

    void set(const std::string& param, std::string value);
    void set(const std::string& param, bool value);
    void set(const std::string& param, char value);
    void set(const std::string& param, int value);
    void set(const std::string& param, float value);


private:
    int findIndex(const std::string& param) const;
    bool read();
    bool write() const;

    bool m_isChanged;
    std::string m_filename;
    std::vector<std::pair<std::string, std::string> > m_data;
    size_t m_size;
};

#endif // SETTINGSPARSER_INCLUDE
```

### SettingsParser.cpp
```cpp
#include "SettingsParser.hpp"

SettingsParser::SettingsParser() : m_size(0), m_isChanged(false)
{

}


SettingsParser::~SettingsParser()
{
    saveToFile();
}


bool SettingsParser::loadFile(const std::string& filename)
{
    m_data.clear();
    m_size = 0;
    m_filename = filename;
    return read();
}


bool SettingsParser::saveToFile() const
{
    if(m_isChanged)
        return write();
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
    std::string line, param, value;
    while(std::getline(in, line))
    {
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
        m_data.push_back(make_pair(param, value));
        m_size++;
    }
    in.close();
    m_isChanged = false;
    return true;
}


bool SettingsParser::write() const
{
    std::ofstream out(m_filename);
    if(!out.is_open())
    {
        std::cerr << "Error: Unable to open settings file: \"" << m_filename << "\" for writing!" << std::endl;
        return false;
    }
    for(size_t i = 0; i < m_size; ++i)
    {
        if(m_data[i].first[0] == '#' || m_data[i].first[0] == 0)
            out << m_data[i].first << std::endl;
        else
            out << m_data[i].first << " = " << m_data[i].second << std::endl;
    }
    out.close();
    return true;
}


bool SettingsParser::isChanged() const
{
    return m_isChanged;
}


int SettingsParser::findIndex(const std::string& param) const
{
    for(unsigned int i = 0; i < m_size; ++i)
    {
        if(m_data[i].first[0] == '#')
            continue;
        if(m_data[i].first == param)
        {
            return i;
        }
    }
    return -1;
}


void SettingsParser::get(const std::string& param, std::string &value) const
{
    int i = findIndex(param);
    if (i > -1)
    {
        value = m_data[i].second;
    }
}


void SettingsParser::get(const std::string& param, bool &value) const
{
    int i = findIndex(param);
    if (i > -1)
    {
        value = ((m_data[i].second == "TRUE") ? true : false);
    }
}


void SettingsParser::get(const std::string& param, char &value) const
{
    int i = findIndex(param);
    if (i > -1)
    {
        value = m_data[i].second[0];
    }
}


void SettingsParser::get(const std::string& param, int &value) const
{
    int i = findIndex(param);
    if (i > -1)
    {
        std::stringstream ss(m_data[i].second);
        ss >> value;
    }
}


void SettingsParser::get(const std::string& param, float &value) const
{
    int i = findIndex(param);
    if (i > -1)
    {
        std::stringstream ss(m_data[i].second);
        ss >> value;
    }
}


void SettingsParser::set(const std::string& param, const std::string value)
{
    int i = findIndex(param);
    if (i > -1)
    {
        m_data[i].second = value;
        m_isChanged =  true;
    }                                 
}


void SettingsParser::set(const std::string& param, const bool value)
{
    int i = findIndex(param);
    if (i > -1)
    {
        m_data[i].second = (value) ? "TRUE" : "FALSE";
        m_isChanged =  true;
    }
}


void SettingsParser::set(const std::string& param, const char value)
{
    int i = findIndex(param);
    if (i > -1)
    {
        std::string tmp = "";
        tmp = value;
        m_data[i].second = tmp;
    }
}


void SettingsParser::set(const std::string& param, const int value)
{
    int i = findIndex(param);
    if (i > -1)
    {
        std::stringstream ss;
        ss << value;
        m_data[i].second = ss.str();
        m_isChanged =  true;
    }
}


void SettingsParser::set(const std::string& param, const float value)
{
    int i = findIndex(param);
    if (i > -1)
    {
        std::stringstream ss;
        ss << value;
        m_data[i].second = ss.str();
        m_isChanged =  true;
    }
}
```