# Audio Player
<a name="top" />
By MickaGL

Works exclusively with the latest revision of SFML 2.

Here is a class using the library [ffmpeg](http://ffmpeg.arrozcru.org/) which allows playback of audio files.  
Works on the same principle that sf:: Music

Inspired by the source code of the tutorial found on this page alFFmpeg : [http://kcat.strangesoft.net/openal.html](http://kcat.strangesoft.net/openal.html)

## <a name="1" />Media.h [ [Top] ](#top)
```cpp
#ifndef MEDIA_H_INCLUDED
#define MEDIA_H_INCLUDED
 
#include <SFML/Audio.hpp>
extern "C"
{
    #include <libavformat/avformat.h>
    #include <libavcodec/avcodec.h>
}
 
namespace sfe
{
class Media : public sf::SoundStream
{
public :
    Media();
    ~Media();
 
    bool OpenFromFile(const std::string &filename);
 
private :
    int GetAVAudioData(void *data, int length);
    void GetNextPacket();
    bool OnGetData(Chunk &data);
    void OnSeek(float timeOffset);
 
    AVFormatContext*    myFormatCtx;
    int                 myAudioStream;
    char*               mySamples;
    char*               myData;
    size_t              myDataSize;
    size_t              myDataSizeMax;
};
 
} // namespace sfe
 
#endif // MEDIA_H_INCLUDED
```

## <a name="2" />Media.cpp [ [Top] ](#top)
```cpp
#include "Media.h"
#include <iostream>
 
namespace sfe
{
Media::Media() :
myFormatCtx     (NULL),
myAudioStream   (-1),
mySamples       (NULL),
myData          (NULL),
myDataSize      (0),
myDataSizeMax   (0)
{
    av_register_all();
}
 
Media::~Media()
{
    Stop();
 
    if (myFormatCtx)
    {
        for (unsigned int i=0; i<myFormatCtx->nb_streams; i++)
        {
            if (myFormatCtx->streams[i]->codec->codec_type == CODEC_TYPE_AUDIO)
            {
                if (myData)
                    av_free(myData);
 
                if (mySamples)
                {
                    delete [] mySamples;
                    mySamples = NULL;
                }
 
                avcodec_close(myFormatCtx->streams[i]->codec);
            }
        }
 
        av_close_input_file(myFormatCtx);
    }
}
 
bool Media::OpenFromFile(const std::string &filename)
{
    if (av_open_input_file(&myFormatCtx, filename.c_str(), NULL, 0, NULL) != 0)
    {
        std::cerr << "Unexisting file!" << std::endl;
        return false;
    }
 
    if (av_find_stream_info(myFormatCtx) < 0)
    {
        std::cerr << "Couldn't find stream information!" << std::endl;
        return false;
    }
 
    for (unsigned int i=0; i<myFormatCtx->nb_streams; i++)
    {
        if (myFormatCtx->streams[i]->codec->codec_type == CODEC_TYPE_AUDIO && myAudioStream < 0)
        {
            AVCodec* codec = avcodec_find_decoder(myFormatCtx->streams[i]->codec->codec_id);
            if (!codec)
                std::cerr << "Unsupported codec!" << std::endl;
            else if (avcodec_open(myFormatCtx->streams[i]->codec, codec) < 0)
                std::cerr << "Couldn't open audio codec context!" << std::endl;
            else
            {
                mySamples = new char[19200];
                if (!mySamples)
                    std::cerr << "Out of memory allocating temp buffer!" << std::endl;
                else
                    myAudioStream = i;
 
                Initialize(myFormatCtx->streams[i]->codec->channels, myFormatCtx->streams[i]->codec->sample_rate);
            }
        }
    }
    if (myAudioStream == -1)
        return false;
 
    return true;
}
 
int Media::GetAVAudioData(void *data, int length)
{
    static size_t decodedDataSize = 0;
    static char decodedData[AVCODEC_MAX_AUDIO_FRAME_SIZE];
    int dec = 0;
 
    while(dec < length)
    {
        if (decodedDataSize > 0)
        {
            size_t rem = length - dec;
            if (rem > decodedDataSize)
                rem = decodedDataSize;
 
            memcpy(data, decodedData, rem);
            data = (char*)data + rem;
            dec += rem;
 
            if (rem < decodedDataSize)
                memmove(decodedData, &decodedData[rem], decodedDataSize - rem);
            decodedDataSize -= rem;
        }
 
        if (decodedDataSize == 0)
        {
            size_t insize = myDataSize;
            if (insize == 0)
            {
                GetNextPacket();
                if (insize == myDataSize)
                    break;
                insize = myDataSize;
                memset(&myData[insize], 0, FF_INPUT_BUFFER_PADDING_SIZE);
            }
 
            int size = AVCODEC_MAX_AUDIO_FRAME_SIZE;
            int len;
            while((len = avcodec_decode_audio2(myFormatCtx->streams[myAudioStream]->codec, (int16_t*)decodedData, &size, (uint8_t*)myData, insize)) == 0)
            {
                if (size > 0)
                    break;
                GetNextPacket();
                if (insize == myDataSize)
                    break;
                insize = myDataSize;
                memset(&myData[insize], 0, FF_INPUT_BUFFER_PADDING_SIZE);
            }
 
            if (len < 0)
                break;
 
            if (len > 0)
            {
                size_t rem = insize - len;
                if (rem)
                    memmove(myData, &myData[len], rem);
                myDataSize = rem;
            }
            decodedDataSize = size;
        }
    }
 
    return dec;
}
 
void Media::GetNextPacket()
{
    AVPacket packet;
    while(av_read_frame(myFormatCtx, &packet) >= 0)
    {
        for (unsigned int i=0; i<myFormatCtx->nb_streams; i++)
        {
            if (myAudioStream == packet.stream_index)
            {
                size_t idx = myDataSize;
                if (idx + packet.size > myDataSizeMax)
                {
                    void *temp = av_realloc(myData, idx+packet.size + FF_INPUT_BUFFER_PADDING_SIZE);
                    if (!temp)
                        return;
                    myData = (char*)temp;
                    myDataSizeMax = idx+packet.size;
                }
 
                memcpy(&myData[idx], packet.data, packet.size);
                myDataSize += packet.size;
 
                return;
            }
            else
                av_free_packet(&packet);
        }
    }
}
 
bool Media::OnGetData(Chunk &data)
{
    if (myAudioStream >= 0)
    {
        int done = GetAVAudioData(mySamples, 19200);
 
        data.Samples   = (sf::Int16*)mySamples;
        data.NbSamples = done/sizeof(sf::Int16);
 
        return (data.NbSamples > 0);
    }
    else
        return false;
}
 
void Media::OnSeek(float timeOffset)
{
 
}
 
} // namespace sfe
```

## <a name="3" />main.cpp [ [Top] ](#top)
```cpp
#include <SFML/Graphics.hpp>
#include "Media.h"
 
int main()
{
    sf::RenderWindow application(sf::VideoMode(640, 480), "ffmpeg avec OpenAL", sf::Style::Close);
 
    sfe::Media media;
    if (!media.OpenFromFile("data/audio.mp3"))
        exit(EXIT_FAILURE);
    media.Play();
 
    while (application.IsOpened())
    {
        sf::Event evenement;
        while (application.GetEvent(evenement))
        {
            if (evenement.Type == sf::Event::Closed)
                application.Close();
 
            if ((evenement.Type == sf::Event::KeyPressed) && (evenement.Key.Code == sf::Key::Escape))
                application.Close();
 
            if ((evenement.Type == sf::Event::KeyPressed) && (evenement.Key.Code == sf::Key::P))
            {
                if (media.GetStatus() != sf::SoundStream::Paused)
                    media.Pause();
                else
                    media.Play();
            }
        }
 
        if (media.GetStatus() != sf::SoundStream::Playing && media.GetStatus() != sf::SoundStream::Paused)
            application.Close();
 
        application.Clear();
 
        application.Display();
    }
 
    return EXIT_SUCCESS;
}
```
