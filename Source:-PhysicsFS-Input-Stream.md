PhysFsStream.hpp:
```C++
#pragma once
#include <SFML/System/InputStream.hpp>
#include <SFML/System/NonCopyable.hpp>

struct PHYSFS_File;

class PhysFsStream : public sf::InputStream, sf::NonCopyable
{
public:
    PhysFsStream(const char * filename = 0x0);
    virtual ~PhysFsStream();
    bool isOpen() const;
    bool open(const char * filename);
    void close();
    virtual sf::Int64 read(void * data, sf::Int64 size);
    virtual sf::Int64 seek(sf::Int64 position);
    virtual sf::Int64 tell();
    virtual sf::Int64 getSize();

private:
    PHYSFS_File * m_File;

};
```

PhysFsStream.cpp:
```C++
#include "PhysFsStream.hpp"
#include <physfs.h>

PhysFsStream::PhysFsStream(const char * filename) : m_File(0x0)
{
    if(filename)
        open(filename);
}

PhysFsStream::~PhysFsStream()
{
    close();
}

bool PhysFsStream::isOpen() const
{
    return (m_File != 0x0);
}

bool PhysFsStream::open(const char * filename)
{
    close();
    m_File = PHYSFS_openRead(filename);
    return isOpen();
}

void PhysFsStream::close()
{
    if(isOpen())
        PHYSFS_close(m_File);
    m_File = 0x0;
}

sf::Int64 PhysFsStream::read(void * data, sf::Int64 size)
{
    if(!isOpen())
        return -1;

    // PHYSFS_read returns the number of 'objects' read or -1 on error.
    // We assume our objects are single bytes and request to read size
    // amount of them.
    return PHYSFS_read(m_File, data, 1, static_cast<PHYSFS_uint32>(size));
}

sf::Int64 PhysFsStream::seek(sf::Int64 position)
{
    if(!isOpen())
        return -1;

    // PHYSFS_seek return 0 on error and non zero on success
    if(PHYSFS_seek(m_File, position))
        return tell();
    else
        return -1;
}

sf::Int64 PhysFsStream::tell()
{
    if(!isOpen())
        return -1;

    // PHYSFS_tell returns the offset in bytes or -1 on error just like SFML wants.
    return PHYSFS_tell(m_File);
}

sf::Int64 PhysFsStream::getSize()
{
    if(!isOpen())
        return -1;

    // PHYSFS_fileLength also the returns length of file or -1 on error just like SFML wants.
    return PHYSFS_fileLength(m_File);
}
```

example.cpp:
```C++
#include <SFML/Graphics.hpp>
#include <physfs.h>
#include "PhysFsStream.hpp"

int main (int argc, char* argv[])
{
    // Initialize PhysicsFS and define a search path.
    PHYSFS_init(argv[0]);
    PHYSFS_addToSearchPath("MyWonderfulArchive.7z", 1);

    // Create a stream and open a file that is inside "MyWonderfulArchive.7z".
    PhysFsStream wonderfullStream;
    wonderfullStream.open("MyWonderfulPictureInArchive.png");

    // Make a sprite from our file.
    sf::Texture tex;
    tex.loadFromStream(wonderfullStream);
    sf::Sprite sprite(tex);
    
    // Create a window and display the sprite
    sf::RenderWindow app(sf::VideoMode(600, 480), "PhysFS");
    while(app.isOpen())
    {
        sf::Event event;
        while (app.pollEvent(event))
            if (event.type == sf::Event::Closed)
                app.close();

        app.clear();
        app.draw(sprite);
        app.display();
    }

    // Finalize PhysicsFS
    PHYSFS_deinit();
}
```