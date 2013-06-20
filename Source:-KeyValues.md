This is a simple class for write and read settings or for other stuff.
Feel free to use it or change it =)

### Simple file to parse

```text

; comment
; window height/width
height=800
width=600

; npc pos
PosX=10
PosY=25

; button label
LabelName=StuffAndThaaaangs

; Player Speed
Speed=10.f

```

### Using example

```

int main(int argc, const char * argv[])
{
    KeyValue keyValue;
    int Height = keyValue.GetInt("height");
    int Width = keyValue.GetInt("width");
    
    std::string ButtonName = keyValue.GetString("LabelName");
    
    float PlayerSpeed = keyValue.GetFloat("Speed");
    
    KeyValue SaveValue;
    SaveValue.SetString("Key", "Name");
    SaveValue.SetInt("PosX", 50);
    SaveValue.SetInt("PosY", 50);
    SaveValue.SetFloat("Speed", 125.f);
}

```


### KeyValues.h

```

//
//  KeyValues.h
//  KeyValues
//
//  Created by David S. on 06.06.13.
//  Copyright (c) 2013 David S. All rights reserved.
//

#ifndef KEYVALUES_H
#define KEYVALUES_H

#include <string>
#include <map>

class KeyValue
{
public:
    KeyValue();
    ~KeyValue();
    
    void LoadFromFile(const std::string &filename);
    void SaveToFile(const std::string &filename);
    
    void SetString(const std::string &key,const std::string &value);
    void SetInt(const std::string &key,int value);
    void SetFloat(const std::string &key, float value);
    
    std::string GetString(const std::string &key);
    int GetInt(const std::string &key);
    float GetFloat(const std::string &key);
    
private:
    void ExtractKey(std::string &key,std::size_t Pos,const std::string &line);
    void ExtractValue(std::string &value,std::size_t Pos,const std::string &line);
    bool KeyExist(const std::string &key) const;
    std::map<std::string,std::string> m_Keys;
};

#endif

```

### KeyValues.cpp

```

//
//  KeyValues.cpp
//  KeyValues
//
//  Created by David S. on 06.06.13.
//  Copyright (c) 2013 David S. All rights reserved.
//

#include "KeyValues.h"
#include <fstream>
#include <iostream>

//---------------------------------------------------------
//---------------------------------------------------------
KeyValue::KeyValue()
{
}

//---------------------------------------------------------
//---------------------------------------------------------
KeyValue::~KeyValue()
{
}

//---------------------------------------------------------
//---------------------------------------------------------
void KeyValue::LoadFromFile(const std::string &filename)
{
    std::ifstream file;
    file.open(filename.c_str());
    
    std::string line;
    std::size_t lineNo = 0;
    std::string Key,Value;
    
    if(file.is_open())
    {
        while (std::getline(file,line))
        {
            lineNo++;
            std::string temp = line;
            
            temp.erase(0,temp.find_first_not_of("\t "));
            std::size_t Pos = temp.find('=');
            
            if(temp.find(';') != std::string::npos)
                continue;
            
            ExtractKey(Key, Pos, temp);
            ExtractValue(Value, Pos, temp);
        
            if(!KeyExist(Key))
                m_Keys[Key] = Value;
            else
                std::cout<<Key<<" already exists."<<std::endl;
        }
    }
    else
    {
        std::cout<<"KeyValues: File "<<filename<<" couldn't be found!\n";
    }
    file.close();
}

//---------------------------------------------------------
//---------------------------------------------------------
void KeyValue::SaveToFile(const std::string &filename)
{
    std::ofstream file;
    file.open(filename.c_str());
    
    if(file.is_open())
    {
        for(std::map<std::string,std::string>::iterator it = m_Keys.begin(); it != m_Keys.end(); ++it)
        {
            file<<(*it).first<<"="<<(*it).second<<std::endl;
        }
    }
    else
    {
        std::cout<<"KeyValues: File "<<filename<<" couldn't be found!\n";
    }
    
    file.close();
}

//---------------------------------------------------------
//---------------------------------------------------------
void KeyValue::SetString(const std::string &key,const std::string &value)
{
    if (!KeyExist(key))
        m_Keys[key] = value;
    else
        std::cout<<key<<" already exists."<<std::endl;
}

//---------------------------------------------------------
//---------------------------------------------------------
void KeyValue::SetInt(const std::string &key, int value)
{
    if (!KeyExist(key))
        m_Keys[key] = std::to_string(value);
    else
        std::cout<<key<<" already exists."<<std::endl;
}

//---------------------------------------------------------
//---------------------------------------------------------
void KeyValue::SetFloat(const std::string &key, float value)
{
    if (!KeyExist(key))
        m_Keys[key] = std::to_string(value);
    else
        std::cout<<key<<" already exists."<<std::endl;
}

//---------------------------------------------------------
//---------------------------------------------------------
std::string KeyValue::GetString(const std::string &key)
{
    std::map<std::string,std::string>::iterator it = m_Keys.find(key);
    if(it != m_Keys.end())
        return (*it).second;
    else
        return "";
}

//---------------------------------------------------------
//---------------------------------------------------------
int KeyValue::GetInt(const std::string &key)
{
    std::map<std::string,std::string>::iterator it = m_Keys.find(key);
    int temp;
    if(it != m_Keys.end())
    {
        temp = std::stoi((*it).second);
        return temp;
    }
    else
        return 0;
}

//---------------------------------------------------------
//---------------------------------------------------------
float KeyValue::GetFloat(const std::string &key)
{
    std::map<std::string,std::string>::iterator it = m_Keys.find(key);
    float temp;
    if(it != m_Keys.end())
    {
        temp = std::stof((*it).second);
        return temp;
    }
    else
        return 0.0f;
}

//---------------------------------------------------------
//---------------------------------------------------------
void KeyValue::ExtractKey(std::string &key, std::size_t Pos, const std::string &line)
{
    key = line.substr(0,Pos);
    if(key.find('\t') != line.npos || key.find(' ') != line.npos)
        key.erase(key.find_first_of("\t "));
}

//---------------------------------------------------------
//---------------------------------------------------------
void KeyValue::ExtractValue(std::string &value, std::size_t Pos, const std::string &line)
{
    value = line.substr(Pos + 1);
    value.erase(0,value.find_first_not_of("\t "));
    value.erase(value.find_last_not_of("\t ") + 1);
}

//---------------------------------------------------------
//---------------------------------------------------------
bool KeyValue::KeyExist(const std::string &key) const
{
    return m_Keys.find(key) != m_Keys.end();
}

```