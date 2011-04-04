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
```

## <a name="3" />main.cpp [ [Top] ](#top)
```cpp
```
