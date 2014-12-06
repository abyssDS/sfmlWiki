#SFML

## Overview
This is simple class for reading and writing settings in a human readable from a text file. It allows you to read a `std::string`, `bool`, `char`, `int` or `float` from a file. You can then access these values from within your program. You may also change the values and write them back to the file. The syntax is basically a `key = value` pair.

Please note that the keys have to be unique. If the same name is used twice, the value will be overwritten.

The original author is cristaloleg, but the class was completely refractored and improved by [Foaly](https://github.com/Foaly). 
It can also be found in [this repository](https://github.com/Foaly/SettingsParser). (be sure to check it out, in case there were any updates, that have not been posted here yet)

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

# physics constants
g = 9.81

# create a rectangle at 10, 10 with width 5, height 10
rectangle = 10, 10, 5, 10

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
#include <vector>

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

    std::vector<int> rectangle;
    settings.get("rectangle", rectangle);
    
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
#include <vector>
#include <sstream>

class SettingsParser 
{
public:
    SettingsParser();
    ~SettingsParser();

    bool loadFromFile(const std::string& filename);
    bool saveToFile();

    bool isChanged() const;
    
    template<typename T>
    void get(const std::string& key, T & value) const;
    template<typename T>
    void get(const std::string& key, std::vector<T> &value) const;
    
    template<typename T>
    void set(const std::string &key, const T value);
    template<typename T>
    void set(const std::string& key, const std::vector<T> value);

    void print() const;

private:
    
    //return the string in the type of T
    template<typename T>
    T convertToType(const std::string &input) const;
    //return string of type T
    template<typename T>
    std::string convertToStr(const T input) const;
    
    bool read();
    bool write() const;
    std::pair<std::string, std::string> parseLine(const std::string &line) const;

    bool m_isChanged;
    std::string m_filename;
    std::map<std::string, std::string> m_data;
    const std::locale m_locale;
};


template<typename T>
inline std::string SettingsParser::convertToStr(T input) const {
    throw "Unsupported type supplied, either change types, or write a correct conversion function for the template type.";
}

template<>
inline std::string SettingsParser::convertToStr<std::string>(std::string value) const {
    return value;
}

template<>
inline std::string SettingsParser::convertToStr<const char*>(const char* value) const {
    return std::string(value);
}

template<>
inline std::string SettingsParser::convertToStr<bool>(bool value) const {
    return (value) ? "TRUE" : "FALSE";
}

template<>
inline std::string SettingsParser::convertToStr<char>(char value) const {
    std::string tmp = "";
    tmp = value;
    return tmp;
}

template<>
inline std::string SettingsParser::convertToStr<int>(int value) const {
    std::stringstream ss;
    ss << value;
    return ss.str();
}

template<>
inline std::string SettingsParser::convertToStr<float>(float value) const {
    std::stringstream ss;
    ss << value;
    return ss.str();
}

template<>
inline std::string SettingsParser::convertToStr<short>(short value) const {
    std::stringstream ss;
    ss << value;
    return ss.str();
}

template<>
inline std::string SettingsParser::convertToStr<double>(double value) const {
    std::stringstream ss;
    ss << value;
    return ss.str();
}

template <typename T>
inline T SettingsParser::convertToType(const std::string &input) const {
    throw "Unconvertable type encountered, please use a different type, or define the handle case in SettingsParser.hpp";
}

template<>
inline int SettingsParser::convertToType<int>(const std::string &input) const {
    int value;
    std::stringstream ss(input);
    ss >> value;
    
    return value;
}

template<>
inline double SettingsParser::convertToType<double>(const std::string &input) const {
    double value;
    std::stringstream ss(input);
    ss >> value;
    
    return value;
}

template<>
inline float SettingsParser::convertToType<float>(const std::string &input) const {
    float value;
    std::stringstream ss(input);
    ss >> value;
    
    return value;
}

template<>
inline short SettingsParser::convertToType<short>(const std::string &input) const {
    short value;
    std::stringstream ss(input);
    ss >> value;
    
    return value;
}

template<>
inline bool SettingsParser::convertToType<bool>(const std::string &input) const {
    return input == "TRUE" ? true : false;
}

template<>
inline char SettingsParser::convertToType<char>(const std::string &input) const {
    return input[0];
}

template<>
inline std::string SettingsParser::convertToType<std::string>(const std::string &input) const {
    return input;
}

template<typename T>
inline void SettingsParser::get(const std::string& key, T &value) const {
    auto it = m_data.find(key);
    
    if (it != m_data.end()){
        value = convertToType<T>(it->second);
    }
}

/**
 * This method tries to read the value of a key into a vector. The values have to be
 * seperated by comma. The vector is cleared before it is filled.
 */
template<typename T>
inline void SettingsParser::get(const std::string& key, std::vector<T> &value) const {
    auto it = m_data.find(key);
    if (it != m_data.end()){
        
        std::string output;
        std::istringstream parser(it->second);
        
        value.clear();
        
        //split by comma
        while (getline(parser, output, ',')){
            value.push_back(convertToType<T>(output));
        }
    }
}

template<typename T>
inline void SettingsParser::set(const std::string& key, const T value) {
    // the [] operator replaces the value if the key is found, if not it creates a new element
    m_data[key] = convertToStr<T>(value);
    m_isChanged = true;
}

template<typename T>
inline void SettingsParser::set(const std::string &key, const std::vector<T> value) {
    // transform the vector into a string that seperates the elements with a comma
    std::string valueAsString;
    for (size_t i = 0; i < value.size() - 1; ++i){
        valueAsString += convertToStr<T>(value.at(i)) + ",";
    }
    valueAsString += convertToStr<T>(value.back());

    // the [] operator replaces the value if the key is found, if not it creates a new element
    m_data[key] = valueAsString;
    m_isChanged = true;
}

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
        std::cerr << "Error: Unable to open settings file \"" << m_filename << "\" for reading!" << std::endl;
        return false;
    }

    std::string line;
    while(std::getline(in, line))
    {
        // parse line
        std::pair<std::string, std::string> keyValuePair = parseLine(line);

        if(!keyValuePair.first.empty())
        {
            // if the line is not empty or a comment save it to the map
            m_data[keyValuePair.first] = keyValuePair.second;
        }
    }

    in.close();
    m_isChanged = false;
    return true;
}


bool SettingsParser::write() const
{
    std::vector<std::pair<std::string, std::string>> fileContents;

    std::ifstream in(m_filename);

    // read the file into a vector and replace the values of the keys that match with our map
    if(in.is_open())
    {
        std::string line;
        while(std::getline(in, line))
        {
            // parse line
            std::pair<std::string, std::string> keyValuePair = parseLine(line);

            if(!keyValuePair.first.empty())
            {
                // check if the key is found in the map
                auto it = m_data.find(keyValuePair.first);
                if (it != m_data.end())
                {
                    // if so take it's value, otherwise the value from the file is kept
                    keyValuePair.second = it->second;
                }
            }
            else
            {
                // if the line is empty or a comment simply take the whole line as the key
                keyValuePair.first = line;
            }
            fileContents.push_back(keyValuePair);
        }
    }
    else
    {
        // Can't open file for reading. Use only the data from the map
        for (auto it = m_data.begin(); it != m_data.end(); ++it)
            fileContents.push_back(std::make_pair(it->first, it->second));
    }

    in.close();


    // open the file for writing
    std::ofstream out(m_filename);
    if(!out.is_open())
    {
        std::cerr << "Error: Unable to open settings file \"" << m_filename << "\" for writing!" << std::endl;
        return false;
    }
    for (auto it = fileContents.begin() ; it != fileContents.end(); ++it)
    {
        out << it->first; // write the key

        if(!it->second.empty())
            // if this line is not empty or a comment also write the assignment and the value
            out << " = " << it->second; 

        out << std::endl;
    }
    out.close();
    return true;
}


/**
 * This method parses a line from our format ("key = value") into a std::pair<std::string, std::string>
 * containing th key and the value.
 * If the line is empty or a comment (starts with a '#') an empty pair is returned.
 */
std::pair<std::string, std::string> SettingsParser::parseLine(const std::string &line) const
{
    if(line.size() > 0 && line[0] != '#')
    {
        size_t index = 0;
        // trim leading whitespace
        while(std::isspace(line[index], m_locale))
            index++;
        // get the key string
        const size_t beginKeyString = index;
        while(!std::isspace(line[index], m_locale) && line[index] != '=')
            index++;
        const std::string key = line.substr(beginKeyString, index - beginKeyString);

        // skip the assignment
        while(std::isspace(line[index], m_locale) || line[index] == '=')
            index++;

        // get the value string
        const std::string value = line.substr(index, line.size() - index);

        // return the key value pair
        return std::make_pair(key, value);
    }

    // if this line is emtpy or a comment, return an empty pair
    return std::make_pair(std::string(), std::string());
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
```