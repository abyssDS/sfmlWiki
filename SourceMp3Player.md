# Mp3 Player
<a name="top" />
By MickaGL

Works exclusively with the latest revision of SFML 2.

Here is a class using the library [mpg123](http://www.mpg123.de/index.shtml) which allows playback of MP3 files.  
Works on the same principle that sf:: Music

Attention, decoding MP3 files is subject to strict licensing and is paid as part of a non-personal use.  
[http://www.mp3licensing.com](http://www.mp3licensing.com)

## <a name="1" />Mp3.h [ [Top] ](#top)
```cpp
#ifndef MP3_H_INCLUDED
#define MP3_H_INCLUDED

#include <SFML/Audio.hpp>
#include "mpg123.h"

namespace sfe
{
class Mp3 : public sf::SoundStream
{
public :
    Mp3();
    ~Mp3();

    bool OpenFromFile(const std::string& filename);

protected :
    bool OnGetData(Chunk& data);
    void OnSeek(float timeOffset);

private :
    mpg123_handle*      myHandle;
    size_t              myBufferSize;
    unsigned char*      myBuffer;
    sf::Mutex           myMutex;
};

} // namespace sfe

#endif // MP3_H_INCLUDED
```

## <a name="2" />Mp3.cpp [ [Top] ](#top)
```cpp
#include "Mp3.h"
#include <iostream>

namespace sfe
{
Mp3::Mp3() :
myHandle    (NULL),
myBufferSize(0),
myBuffer    (NULL)
{

}

Mp3::~Mp3()
{
    Stop();

    if (myBuffer)
    {
        delete [] myBuffer;
        myBuffer = NULL;
    }

    mpg123_close(myHandle);
    mpg123_delete(myHandle);
    mpg123_exit();
}

bool Mp3::OpenFromFile(const std::string& filename)
{
    Stop();

    int  err = MPG123_OK;
    if ((err = mpg123_init()) != MPG123_OK)
    {
        std::cerr << mpg123_plain_strerror(err) << std::endl;
        return false;
    }

    myHandle = mpg123_new(NULL, &err);
    if (!myHandle)
    {
        std::cerr << "Unable to create mpg123 handle: " << mpg123_plain_strerror(err) << std::endl;
        return false;
    }

    if (mpg123_open(myHandle, filename.c_str()) != MPG123_OK)
    {
        std::cerr << mpg123_strerror(myHandle) << std::endl;
        return false;
    }

    long rate = 0;
    int  channels = 0, encoding = 0;
    if (mpg123_getformat(myHandle, &rate, &channels, &encoding) != MPG123_OK)
    {
        std::cerr << "Failed to get format information for \"" << filename << "\"" << std::endl;
        return false;
    }

    myBufferSize = mpg123_outblock(myHandle);
    myBuffer = new unsigned char[myBufferSize];
    if (!myBuffer)
    {
        std::cerr << "Failed to reserve memory for decoding one frame for \"" << filename << "\"" << std::endl;
        return false;
    }

    Initialize(channels, rate);

    return true;
}

bool Mp3::OnGetData(Chunk& data)
{
    sf::Lock lock(myMutex);

    if (myHandle)
    {
        size_t done;
        mpg123_read(myHandle, myBuffer, myBufferSize, &done);

        data.Samples   = (short*)myBuffer;
        data.NbSamples = done/sizeof(short);

        return (data.NbSamples > 0);
    }
    else
        return false;
}

void Mp3::OnSeek(float timeOffset)
{
    sf::Lock lock(myMutex);

    if (myHandle)
        mpg123_seek(myHandle, timeOffset, 0);
}

} // namespace sfe
```

## <a name="3" />main.cpp [ [Top] ](#top)
```cpp
#include <SFML/Graphics.hpp>
#include "Mp3.h"

int main()
{
    sf::RenderWindow application(sf::VideoMode::GetDesktopMode(), "", sf::Style::Fullscreen);

    sfe::Mp3 musique;
    if (!musique.OpenFromFile("music.mp3"))
        exit(EXIT_FAILURE);
    musique.Play();

    while (application.IsOpened())
    {
        sf::Event evenement;
        while (application.GetEvent(evenement))
        {
            if ((evenement.Type == sf::Event::KeyPressed) && (evenement.Key.Code == sf::Key::Escape))
                application.Close();

            if ((evenement.Type == sf::Event::KeyPressed) && (evenement.Key.Code == sf::Key::P))
            {
                if (musique.GetStatus() != sf::SoundStream::Paused)
                    musique.Pause();
                else
                    musique.Play();
            }
        }

        if (musique.GetStatus() != sf::SoundStream::Playing && musique.GetStatus() != sf::SoundStream::Paused)
            application.Close();

        application.Clear();

        application.Display();
    }

    return EXIT_SUCCESS;
}
```
