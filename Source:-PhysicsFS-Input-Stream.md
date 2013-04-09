```
#include <iostream>
#include <physfs/physfs.h>
#include <SFML/Graphics.hpp>

class PhysFsStream : public sf::InputStream
{
public:

    PhysFsStream(const char* filename=0x0) :
        m_File(0x0)
    {
        if (filename)
            open(filename);
    }

    virtual ~PhysFsStream()
    {
        close();
    }

    bool isOpen() const
    {
        return (m_File != 0x0);
    }

    bool open(const char* filename)
    {
        close();
        m_File = PHYSFS_openRead(filename);
        return isOpen();
    }

    void close()
    {
        if (isOpen())
            PHYSFS_close(m_File);
    }

    virtual sf::Int64 read(void* data, sf::Int64 size)
    {
        if (!isOpen())
            return -1;

        // PHYSFS_read returns the number of 'objects' read or -1 on error.
        // We assume our objects are single bytes and request to read size
        // amount of them.
        return PHYSFS_read(m_File, data, 1, static_cast<PHYSFS_uint32>(size));
    }

    virtual sf::Int64 seek(sf::Int64 position)
    {
        if (!isOpen())
            return -1;

        // PHYSFS_seek return 0 on error and non zero on success
	if (PHYSFS_seek(m_File, position))
            return tell();
        else
            return -1;
    }

    virtual sf::Int64 tell()
    {
        if (!isOpen())
            return -1;

        // PHYSFS_tell returns the offset in bytes or -1 on error just like SFML wants.
        return PHYSFS_tell(m_File);
    }

    virtual sf::Int64 getSize()
    {
        if (!isOpen())
            return -1;

        // PHYSFS_fileLength also the returns length of file or -1 on error just like SFML wants.
        return PHYSFS_fileLength(m_File);
    }

private:
    PHYSFS_File * m_File;

};
```

```
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