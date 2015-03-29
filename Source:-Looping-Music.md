This is a demonstration of how one could implement a looping music class based on sf::SoundStream using sf::InputSoundFile. It is an alternate implementation of what was [#629](https://github.com/LaurentGomila/SFML/pull/629) that is built on top of SFML without requiring any modifications to SFML itself. Anyone can use this in their project by simply copying the code over. This class is very rudimentary, possibly missing many features that one might need, but it should be relatively simple to extend it in many ways.

```cpp

////////////////////////////////////////////////////////////
// Headers
////////////////////////////////////////////////////////////
#include <SFML/Audio.hpp>
#include <iomanip>
#include <iostream>

class LoopingMusic : public sf::SoundStream
{
public:

    ////////////////////////////////////////////////////////////
    /// \brief Default constructor
    ///
    ////////////////////////////////////////////////////////////
    LoopingMusic(sf::InputSoundFile& file) :
    m_file       (file),
    m_loopBegin  (0),
    m_loopEnd    (0),
    m_loopCurrent(0)
    {
        // Resize the internal buffer so that it can contain 1 second of audio samples
        m_samples.resize(m_file.getSampleRate() * m_file.getChannelCount());

        // Initialize the stream
        SoundStream::initialize(m_file.getChannelCount(), m_file.getSampleRate());
    }

    ////////////////////////////////////////////////////////////
    /// \brief Destructor
    ///
    ////////////////////////////////////////////////////////////
    ~LoopingMusic()
    {
        // We must stop before destroying the file
        stop();
    }

    ////////////////////////////////////////////////////////////
    /// \brief Get the position of the beginning of the sound's looping sequence
    ///
    /// \return Loop begin position
    ///
    /// \see getLoopEnd, setLoopPoints
    ///
    ////////////////////////////////////////////////////////////
    sf::Time getLoopBegin() const
    {
        unsigned int sampleRate = getSampleRate();
        unsigned int channelCount = getChannelCount();

        if (sampleRate && channelCount)
            return sf::seconds(static_cast<float>(m_loopBegin) / sampleRate / channelCount);

        return sf::Time::Zero;
    }

    ////////////////////////////////////////////////////////////
    /// \brief Get the position of the end of the sound's looping sequence
    ///
    /// \return Loop end position
    ///
    /// \see getLoopBegin, setLoopPoints
    ///
    ////////////////////////////////////////////////////////////
    sf::Time getLoopEnd() const
    {
        unsigned int sampleRate = getSampleRate();
        unsigned int channelCount = getChannelCount();

        if (sampleRate && channelCount)
            return sf::seconds(static_cast<float>(m_loopEnd) / sampleRate / channelCount);

        return sf::Time::Zero;
    }

    ////////////////////////////////////////////////////////////
    /// \brief Sets the beginning and end of the sound's looping sequence using sf::Time
    ///
    /// Loop points allow one to specify a pair of positions such that, when the music
    /// is enabled for looping, it will seamlessly seek to begin whenever it encounters end.
    /// The input values are clamped to the duration of the sound. If they are the same,
    /// then a closed loop cannot be formed, and this function will "reset" the loop to the
    /// full length of the sound. Note that the implicit "loop points" from the end to the
    /// beginning of the stream are still honored. Because of this, "reverse" loop ranges,
    /// where end comes before begin, are allowed, and will cause the sound to loop everything
    /// "outside" of the specified range. This function can be safely called at any point
    /// after a stream is opened, and will be applied to a playing sound without affecting
    /// the current playing offset.
    ///
    /// \param begin    The offset of the beginning of the loop. Can be any time point within the sound's length
    /// \param end      The offset of the end of the loop. If Time::Zero is passed, it defaults to the end of the sound
    ///
    /// \see getLoopBegin, getLoopEnd
    ///
    ////////////////////////////////////////////////////////////
    void setLoopPoints(sf::Time begin, sf::Time end)
    {
        std::size_t sampleCount = m_file.getSampleCount();

        // Reset to playing whole stream
        if (begin == end)
        {
            m_loopBegin = 0;
            m_loopEnd = sampleCount;
            return;
        }

        unsigned int sampleRate = getSampleRate();
        unsigned int channelCount = getChannelCount();

        begin = sf::microseconds(std::max(begin.asMicroseconds(), 0ll));
        end = sf::microseconds(std::max(end.asMicroseconds(), 0ll));

        sf::Uint64 loopBegin = static_cast<sf::Uint64>(begin.asSeconds() * sampleRate * channelCount);
        sf::Uint64 loopEnd = static_cast<sf::Uint64>(end.asSeconds() * sampleRate * channelCount);

        loopBegin = std::min(loopBegin, static_cast<sf::Uint64>(sampleCount));
        loopEnd = std::min(loopEnd, static_cast<sf::Uint64>(sampleCount));

        if (!loopEnd)
            loopEnd = sampleCount;

        loopBegin -= loopBegin % channelCount;
        loopEnd -= loopEnd % channelCount;

        m_loopBegin = loopBegin;
        m_loopEnd = loopEnd;
    }

protected:

    ////////////////////////////////////////////////////////////
    /// \brief Request a new chunk of audio samples from the stream source
    ///
    /// This function fills the chunk from the next samples
    /// to read from the audio file.
    ///
    /// \param data Chunk of data to fill
    ///
    /// \return True to continue playback, false to stop
    ///
    ////////////////////////////////////////////////////////////
    virtual bool onGetData(Chunk& data)
    {
        sf::Lock lock(m_mutex);

        // Fill the chunk parameters
        data.samples     = &m_samples[0];
        data.sampleCount = static_cast<std::size_t>(m_file.read(&m_samples[0], m_samples.size()));

        if (!data.sampleCount)
        {
            // Check if we have reached the end of the audio file
            return data.sampleCount == m_samples.size();
        }

        // Standard non-looping case
        if (m_loopBegin == m_loopEnd || !getLoop())
        {
            m_loopCurrent += data.sampleCount;

            // Check if we have reached the end of the audio file
            return data.sampleCount == m_samples.size();
        }

        // We are looping

        // Check for a loop transition
        if (m_loopCurrent + data.sampleCount > m_loopEnd)
        {
            std::size_t endSampleCount = std::min(static_cast<std::size_t>(m_loopEnd - m_loopCurrent), data.sampleCount);
            std::size_t beginSampleCount = m_samples.size() - endSampleCount;

            // Jump back to the beginning of the sequence
            m_file.seek(m_loopBegin);

            // Fill the rest of the buffer with the data at the beginning
            beginSampleCount = static_cast<std::size_t>(m_file.read(&m_samples[endSampleCount], beginSampleCount));

            data.sampleCount = endSampleCount + beginSampleCount;
            m_loopCurrent = m_loopBegin + beginSampleCount;
        }
        else
        {
            m_loopCurrent += data.sampleCount;
        }

        return data.sampleCount == m_samples.size();
    }

    ////////////////////////////////////////////////////////////
    /// \brief Change the current playing position in the stream source
    ///
    /// \param timeOffset New playing position, from the beginning of the music
    ///
    ////////////////////////////////////////////////////////////
    virtual void onSeek(sf::Time timeOffset)
    {
        sf::Lock lock(m_mutex);

        sf::Uint64 sampleOffset = static_cast<sf::Uint64>(timeOffset.asSeconds() * getSampleRate() * getChannelCount());
        sampleOffset -= sampleOffset % getChannelCount();

        m_loopCurrent = sampleOffset;
        m_file.seek(sampleOffset);
    }

private:

    ////////////////////////////////////////////////////////////
    // Member data
    ////////////////////////////////////////////////////////////
    sf::InputSoundFile&    m_file;        ///< The streamed music file
    sf::Time               m_duration;    ///< Music duration
    std::vector<sf::Int16> m_samples;     ///< Temporary buffer of samples
    sf::Mutex              m_mutex;       ///< Mutex protecting the data
    sf::Uint64             m_loopBegin;   ///< First sample position
    sf::Uint64             m_loopEnd;     ///< Last sample position
    sf::Uint64             m_loopCurrent; ///< Current sample position
};

////////////////////////////////////////////////////////////
/// Play a looping music with loop points
///
////////////////////////////////////////////////////////////
void playMusicLoop()
{
    // Load an ogg music file
    sf::InputSoundFile musicFile;
    if (!musicFile.openFromFile("resources/orchestral.ogg"))
        return;

    // Create our looping music using the music file
    LoopingMusic music(musicFile);

    // Setup the loop points
    music.setLoopPoints(sf::seconds(5.579f), sf::seconds(13.496f));
    float duration = 13.496f - 5.579f;

    // Display sound informations
    std::cout << "orchestral.ogg (Music, Loop):" << std::endl;
    std::cout << " " << musicFile.getDuration().asSeconds() << " seconds"       << std::endl;
    std::cout << " " << musicFile.getSampleRate()           << " samples / sec" << std::endl;
    std::cout << " " << musicFile.getChannelCount()         << " channels"      << std::endl;
    std::cout << " Looping from " << music.getLoopBegin().asSeconds()       <<
                 " sec to " << music.getLoopEnd().asSeconds() << " sec"     << std::endl;

    music.setLoop(true);
    music.play();

    // Loop while the sound is playing. Make sure it stops after a few loops.
    int loops = 3;
    while (music.getStatus() == sf::Sound::Playing)
    {
        // Leave some CPU time for other processes
        sf::sleep(sf::milliseconds(100));
        float time = music.getPlayingOffset().asSeconds();
        if (time >= 13.496f + 3.f * duration)
        {
            loops = 0;
            music.stop();
        }
        else if (time >= 13.496f + 2.f * duration)
        {
            loops = 1;
        }
        else if (time >= 13.496f + 1.f * duration)
        {
            loops = 2;
        }

        // Display the playing position
        std::cout << "\rPlaying... " << std::fixed << std::setprecision(2) << time << " sec with " << loops << " loops to go        ";
        std::cout << std::flush;
    }
    std::cout << std::endl << std::endl;
}


////////////////////////////////////////////////////////////
/// Entry point of application
///
/// \return Application exit code
///
////////////////////////////////////////////////////////////
int main()
{
    // Play a looping music with loop points
    playMusicLoop();

    // Wait until the user presses 'enter' key
    std::cout << "Press enter to exit..." << std::endl;
    std::cin.ignore(10000, '\n');

    return EXIT_SUCCESS;
}
```