# Mp3 Soundfile Reader
<a name="top" />
By hansiC

Built and tested using SFML 2.3

Here is a class using the library [mpg123](http://www.mpg123.de/index.shtml) which allows playback of MP3 files.  
Uses the sf::SoundFileFactory::registerReader() function to register the reader for Mp3 files.

View on github: [https://github.com/hansiC/sfml-soundfileReader-example](https://github.com/hansiC/sfml-soundfileReader-example)

Attention, decoding MP3 files is subject to strict licensing and is paid as part of a non-personal use.  
[http://www.mp3licensing.com](http://www.mp3licensing.com)

## <a name="1" />SoundFileReaderMp3.hpp [ [Top] ](#top)
```cpp

#pragma once

#include <mpg123.h>
#include <SFML/Audio.hpp>

#include <string>
#include <cstdint>

namespace audio {

class SoundFileReaderMp3 : public sf::SoundFileReader
{
public:
    SoundFileReaderMp3();

    ~SoundFileReaderMp3();

	// \brief Check if this reader can handle a file given by an input stream
	// \param stream Source stream to check
	// \return True if the file is supported by this reader
	static bool check(sf::InputStream& stream);

    // \brief Open a sound file for reading
    // \param stream Source stream to read from
    // \param info   Structure to fill with the properties of the loaded sound
    // \return True if the file was successfully opened
	bool open(sf::InputStream& stream, sf::SoundFileReader::Info& info) override;

    // \brief Change the current read position to the given sample offset
    // If the given offset exceeds to total number of samples,
    // this function must jump to the end of the file.
    // \param sampleOffset Index of the sample to jump to, relative to the beginning
    void seek(sf::Uint64 sampleOffset) override;

    // \brief Read audio samples from the open file
    // \param samples  Pointer to the sample array to fill
    // \param maxCount Maximum number of samples to read
    // \return Number of samples actually read (may be less than \a maxCount)
    sf::Uint64 read(sf::Int16* samples, sf::Uint64 maxCount) override;


protected:

    // Encapsulates mp3 output format
    class OutputFormat {
    private:
    	long m_rate = 0;
		int m_channels = 0;
		int m_encoding = 0;
    public:
		bool update (mpg123_handle* handle ){
			return mpg123_getformat(handle, &m_rate, &m_channels, &m_encoding) == MPG123_OK;
		}
		std::string toString() const {
			std::string result;
			result += "format: sample rate in Hz: " + std::to_string(m_rate);
			result += "\nformat: channels: " + std::to_string(m_channels);
			result += "\nformat: encoding: " + std::to_string(m_encoding);
			return result;
		}
    };

    // Easy access to mpg123_frameinfo struct
    class FrameInfo {
    private:
    	mpg123_frameinfo  m_info;
    	const std::size_t m_headerSize = 4;
    public:
    	bool update (mpg123_handle* handle ){
    		return mpg123_info(handle, &m_info) == MPG123_OK;
    	}
    	std::size_t getFrameSizeInBytesIncludingHeader() const {
    		return m_info.framesize;
    	}
    	std::size_t getFrameSizeInBytesNoHeader() const {
			return m_info.framesize - m_headerSize;
		}
    	std::size_t getExpectedNumberOfSamples() const {
    		return getFrameSizeInBytesNoHeader() / sizeof(short);
    	}
    	std::uint32_t getSamplingRateInHz() const {
    		return m_info.rate;
    	}
    	std::uint32_t getNumberOfChannels() const {
			switch (m_info.mode)
			{
				case mpg123_mode::MPG123_M_MONO:
					return 1;
				case mpg123_mode::MPG123_M_STEREO:
				case mpg123_mode::MPG123_M_JOINT:
				case mpg123_mode::MPG123_M_DUAL:
					return 2;
				default:
					break;
			}
			return 0;
    	}
    	std::string toString() const {
    		std::string result;
    		result += "header: MPEG version: ";
    		switch (m_info.version)
    		{
    			case mpg123_version::MPG123_1_0:
    				result += "1.0";
    				break;
    			case mpg123_version::MPG123_2_0:
    				result += "2.0";
    				break;
    			case mpg123_version::MPG123_2_5:
    				result += "2.5";
    				break;
    			default:
    				result += "unknown version";
    				break;
    		}
    		result += "\nheader: MPEG Audio mode: ";
    		switch (m_info.mode)  // Only the mono mode has 1 channel, the others have 2 channels.
    		{
    			case mpg123_mode::MPG123_M_STEREO:
    				result += "Standard Stereo (2 channels)";
    				break;
    			case mpg123_mode::MPG123_M_JOINT:
    				result += "Joint Stereo (2 channels)";
    				break;
    			case mpg123_mode::MPG123_M_DUAL:
    				result += "Dual Channel (2 channels)";
    				break;
    			case mpg123_mode::MPG123_M_MONO:
    				result += "Mono (single channel)";
    				break;
    			default:
    				result += "unknown mode";
    				break;
    		}
    	//	enum mpg123_flags 	    flags
    		result += "\nheader: Mode type of variable bitrate (vbr): ";
    		switch (m_info.vbr)
    		{
    			case mpg123_vbr::MPG123_CBR:
    				result += "Constant Bitrate Mode (default)";
    				break;
    			case mpg123_vbr::MPG123_VBR:
    				result += "Variable Bitrate Mode";
    				break;
    			case mpg123_vbr::MPG123_ABR:
    				result += "Average Bitrate Mode";
    				break;
    			default:
    				result += "unknown vbr mode";
    				break;
    		}
    		result += "\nheader: MPEG Audio Layer (MP1/MP2/MP3): " + std::to_string(m_info.layer);
    		result += "\nheader: Mode extension bit flag: " + std::to_string(m_info.mode_ext);
    		result += "\nheader: Emphasis type (?): " + std::to_string(m_info.emphasis);
    		result += "\nheader: Target average bitrate: " + std::to_string(m_info.abr_rate);
    		result += "\nheader: Bitrate of the frame (kbps): " + std::to_string(m_info.bitrate);
    		result += "\nheader: Sampling rate in Hz: " + std::to_string(m_info.rate);
    		result += "\nheader: Size of the frame (in bytes, including header): " + std::to_string(m_info.framesize);

    		return result;
    	}
    };

    // library objects
    mpg123_handle*      m_handle;
    sf::InputStream*    m_stream;
    FrameInfo           m_frameInfo;
    OutputFormat        m_outputFormat;

    // buffers
    std::size_t         m_streambufferSize;
    unsigned char*      m_streambuffer;

    // Creates the handle to the mp3 library
    bool initializeLibrary();
    // Used by open() to consume just enough data to retrieve frame info
    // and output format so we can set parameters in sf::SoundFileReader::Info
    bool probeFirstFrame(sf::InputStream& stream);
    void fillSfmlInfo(sf::SoundFileReader::Info& info) const;
    // Called by destructor to close the library and delete the buffers
    void close();
};

} // namespace audio
```

## <a name="2" />SoundFileReaderMp3.cpp [ [Top] ](#top)
```cpp
#include "SoundFileReaderMp3.hpp"

#include <cassert>
#include <iostream>
#include <string>
#include <vector>
#include <cstdint>
#include <sstream>
#include <bitset>

namespace audio {

SoundFileReaderMp3::SoundFileReaderMp3()
	: m_handle            (nullptr)
	, m_stream            (nullptr)
	, m_frameInfo         ()
	, m_outputFormat      ()
	, m_streambufferSize  (80000) // any size will work
	, m_streambuffer      (new unsigned char[m_streambufferSize])
{}

SoundFileReaderMp3::~SoundFileReaderMp3()
{
	close();
}

void SoundFileReaderMp3::close() {
	if(m_handle) {
		mpg123_close(m_handle);
		mpg123_delete(m_handle);
		mpg123_exit();
		m_handle = nullptr;
	}
    if (m_streambuffer) {
        delete [] m_streambuffer;
        m_streambuffer = nullptr;
    }
    std::cout << "Mp3 reader close" << std::endl;
}

// \brief Open a sound file for reading
// \param stream Source stream to read from
// \param info   Structure to fill with the properties of the loaded sound
// \return True if the file was successfully opened
bool SoundFileReaderMp3::open(sf::InputStream& stream, sf::SoundFileReader::Info& info) {
	std::cout << "OPEN" << std::endl;
	initializeLibrary();
	probeFirstFrame(stream);

	 if(!m_outputFormat.update(m_handle)){
		std::cout << "Failed to get format information" << std::endl;
		return false;
	}
	std::cout << m_outputFormat.toString() << std::endl;

	if(!m_frameInfo.update(m_handle)){
		std::cout << "Failed to get header information" << std::endl;
		return false;
	}
	std::cout << m_frameInfo.toString() << std::endl;

	fillSfmlInfo(info);

	std::cout << "wrote info: channels: " << info.channelCount << ", expected number of samples: " << info.sampleCount
			  << ", sampling rate: " << info.sampleRate << std::endl;

	// save the reference since we never get it again
	m_stream = &stream;

	std::cout << "OPEN: SUCCESS" << std::endl;
	return true;
}

// calls mpg123_init(), mpg123_new() and mpg123_open_feed
// and does error handling
bool SoundFileReaderMp3::initializeLibrary() {
	if(m_handle) {
		std::cerr << "library already initialized" << std::endl;
		return false;
	}
	// Initialise the mpg123 library.
	// This function is not thread-safe. Call it exactly once per process,
	// before any other (possibly threaded) work with the library.
	int  ok = mpg123_init();
	if (ok != MPG123_OK) {
		std::cerr << "mpg123_init(): " << mpg123_plain_strerror(ok) << std::endl;
		return false;
	}

	// create handle with optional choice of decoder
	m_handle = mpg123_new(NULL, &ok);
	if (!m_handle) {
		std::cerr << "Unable to create mpg123 handle: " << mpg123_plain_strerror(ok) << std::endl;
		return false;
	}
	std::cout << "created handle at address " << (std::uintptr_t)m_handle << std::endl;

	// Later we have to read the audio input from sf::InputStream, which can be from a file,
	// from the network or from any other input stream. This decides how we open mpg123:
	ok = mpg123_open_feed(m_handle);
	if (ok != MPG123_OK) {
		std::cerr << "mpg123_open_feed(): " << mpg123_plain_strerror(ok) << std::endl;
		return false;
	}
	std::cout << "opened handle for feeding" << std::endl;
	return true;
}

bool SoundFileReaderMp3::probeFirstFrame(sf::InputStream& stream) {
	// select a rather small buffer size. we don't want to read actual payload data.
	// we only want to read till the mp3 library can extract header information
	std::size_t streamSize =  2048;
	unsigned char streamBuffer[streamSize];

	std::size_t totalReadBytes = 0;
	std::size_t totalDecodedBytes = 0;
	std::size_t numReadsFromStream = 0;

	for(bool needMore = true; needMore;) {
		sf::Int64 bytesReadFromStream = stream.read(streamBuffer, streamSize); // read from input stream
		numReadsFromStream++;
		if (bytesReadFromStream <= 0) {
			std::cout << (bytesReadFromStream == 0 ? "stream empty" : ("stream error: " + bytesReadFromStream)) << std::endl;
			break;
		}
		totalReadBytes += bytesReadFromStream;

		// feed until happy, do not touch decoded data. we only want to get header information
		std::size_t numDecodedBytes = 0;
		int ret = mpg123_decode(m_handle, streamBuffer, bytesReadFromStream, nullptr, 0, &numDecodedBytes);
		totalDecodedBytes += numDecodedBytes;

		needMore = ret == MPG123_NEED_MORE;
	}
	std::cout << "first header decoded"  << std::endl;
	std::cout << "buffersize: "          << streamSize << std::endl;
	std::cout << "total bytes read: "    << totalReadBytes << std::endl;
	std::cout << "total bytes decoded: " << totalDecodedBytes << std::endl;
	std::cout << "reads from stream: "   << numReadsFromStream << std::endl;
}

void SoundFileReaderMp3::fillSfmlInfo(sf::SoundFileReader::Info& info) const {
	info.channelCount = m_frameInfo.getNumberOfChannels();
	info.sampleCount = m_frameInfo.getExpectedNumberOfSamples();
	info.sampleRate = m_frameInfo.getSamplingRateInHz();
}

// \brief Read audio samples from the open file
// \param samples  Pointer to the sample array to fill
// \param maxCount Maximum number of samples to read
// \return Number of samples actually read. If less samples than maxCount are read, sfml assumes that EOF has been reached and terminates playback.
sf::Uint64 SoundFileReaderMp3::read(sf::Int16* samples, sf::Uint64 maxCount) {
	std::cout << "READ: request up to " << maxCount << " samples. size of streambuffer: " << m_streambufferSize << " bytes" << std::endl;

	// since we get fed bytes from the input stream, casting the
	// output buffer to bytes makes life easier
	unsigned char* byteSamples = (unsigned char*) samples;
	// adjust the requested length
	std::size_t maxBytes = maxCount * sizeof(short);
	std::size_t offset = 0;

	std::size_t totalReadBytes = 0;

	// README: How the sf Audio playback loop works:
	// sf::SoundFileReader::read() gets called in a loop.
	// sf::Soundstream breaks the loop if the return value is 0
	// or smaller than the requested number of samples, maxCount.

	// This is not the main decoding loop but a clean-up loop that consumes any input
	// left from the previous call. This happens if the sample buffer was almost full
	// but the decoder needed more raw input. We read from the stream again and feed
	// everything to the decoder. The decoder is only allowed to fill the small amount
	// left in the sample buffer and has to keep the remaining input.
	// Now is the time to consume that remaining input.
	int ret = 0;
	do
	{
		std::size_t numDecodedBytes = 0;
		ret = mpg123_decode(m_handle,
				  nullptr      // raw input buffer: since we want to consume the remaining input, we don't provide any new one
				, 0            // length of input:  same reason
				, &byteSamples[offset] // array of decoded bytes at sample offset
			    , maxBytes-offset      // number of decoded bytes we want to get out: this calculation is why casting to byte size was a good idea
				, &numDecodedBytes);         // this tells us the size of the decoded data. it will never exceed the size we requested in the above parameter
		offset += numDecodedBytes;     // set sample offset for next call
		std::cout << "decoded " << numDecodedBytes << " bytes from leftover input. offset into samples: " << offset << " / " << maxBytes << std::endl;

	} // NEED_MORE is returned when no more input is available and the output array is not full yet. not sure about ERR though. probably when output is full.
	while (ret != MPG123_NEED_MORE && ret != MPG123_ERR && offset < maxBytes);

	while (ret == MPG123_NEED_MORE && ret != MPG123_ERR && offset < maxBytes)
	{
		// now run the normal reading-decoding loop
		sf::Int64 bytesReadFromStream = m_stream->read(m_streambuffer, m_streambufferSize);
		std::cout << "read " << bytesReadFromStream << " bytes from stream" << std::endl;
		if (bytesReadFromStream <= 0) {
			std::cout << (bytesReadFromStream == 0 ? "stream empty" : ("stream error: " + bytesReadFromStream)) << std::endl;
			break;
		}
		totalReadBytes += bytesReadFromStream;

		// feed the input to the decoder
		std::size_t numDecodedBytes = 0;
		ret = mpg123_decode(m_handle, m_streambuffer, bytesReadFromStream, &byteSamples[offset], maxBytes-offset, &numDecodedBytes);
		offset += numDecodedBytes;
		std::cout << "decoded " << bytesReadFromStream << " to " << numDecodedBytes << " decoded bytes. total decoded bytes: "
				  << offset << " / " << maxBytes << std::endl;
	}

	std::size_t totalSamples = offset/sizeof(short);

	std::cout << "samples delivered: " << totalSamples << " of " << maxCount << " requested"<< std::endl;
	std::cout << "total bytes read: " << totalReadBytes << std::endl;
	std::cout << "READ: SUCCESS" << std::endl;

	return totalSamples;
}

// \brief Change the current read position to the given sample offset
// If the given offset exceeds the total number of samples,
// this function must jump to the end of the file.
// \param sampleOffset Index of the sample to jump to, relative to the beginning
void SoundFileReaderMp3::seek(sf::Uint64 sampleOffset) {
	std::cout << "SEEK sample offset " << sampleOffset << " FIXME not implemented" << std::endl;

	// probably should go something like this
//	off_t ok = mpg123_feedseek (m_handle, sampleOffset, SEEK_SET, NULL);
}

// Quick check if the reader can handle the input
bool SoundFileReaderMp3::check(sf::InputStream& stream) {
	std::cout << "CHECK: can Mp3 Reader handle the stream?" << std::endl;
	SoundFileReaderMp3 reader;
	bool result = reader.initializeLibrary();
	if(result)
		result = reader.probeFirstFrame(stream);
	if(result)
		result = reader.m_outputFormat.update(reader.m_handle);
	if(result)
		result = reader.m_frameInfo.update(reader.m_handle);

	std::cout << "CHECK: " << (result ? "SUCCESS" : "FAILED") << std::endl;

	// no cleanup required since destructor of reader takes care of everything
	return result;
}

} // namespace audio
```

## <a name="3" />Main.cpp [ [Top] ](#top)
```cpp
#include "SoundFileReaderMp3.hpp"

#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>

#include <iostream>
#include <string>
#include <vector>
#include <cstdint>
#include <sstream>
#include <fstream>

std::string getResourcePath(const std::string& executableDir, const std::string& resourceRelativeName) {
	// windows users may have to change "/" for "\"
	std::size_t pos = executableDir.find_last_of("/");
	std::string prefix = executableDir.substr (0, pos+1);
	return prefix + resourceRelativeName;
}

void togglePlayPause(sf::Music& music) {
	sf::SoundSource::Status musicStatus = music.getStatus();
	if(musicStatus == sf::SoundSource::Status::Paused) {
		music.play();
		std::cout << "music play" << std::endl;
	} else if(musicStatus == sf::SoundSource::Status::Playing) {
		music.pause();
		std::cout << "music pause" << std::endl;
	}
}

int main(int argc, char** argv) {
	if(argc < 2) {
		std::cout << argv[0] << ": missing argument <absolute filename>" << std::endl;
		return EXIT_FAILURE;
	}

	std::string filename(argv[1]);

	std::cout << "START: " << filename << std::endl;
	// Register our custom sf::SoundFileReader for mp3 files.
	// This is the preferred way of adding support for a new audio format.
	// Other formats will be handled by their respective readers.
	sf::SoundFileFactory::registerReader<audio::SoundFileReaderMp3>();

	// Load a music to play
	sf::Music music;
	if (!music.openFromFile(filename)) {
		std::cout << "check your file path. also only wav, flac, ogg and mp3 are supported." << std::endl;
		return EXIT_FAILURE;
	}

	// Now we are sure we can play the file
	// Set up the main window

	sf::Sprite sprite;
	sf::Texture texture;
	sf::Vector2u dimensions;

	// See if we can find the image
	std::string textureName = getResourcePath(std::string(argv[0]), "MP3_file_structure.png");
	std::ifstream ifile(textureName);
	if (ifile) {
		texture.loadFromFile(textureName);
		sprite.setTexture(texture);
		dimensions = texture.getSize();
	} else {
		std::cout << "Resource not found" << std::endl;
		dimensions.x = 800;
		dimensions.y = 600;
	}

	// Create the main window
	sf::RenderWindow window(sf::VideoMode(dimensions.x, dimensions.y), "SFML window");

	// Start the music
	music.play();

	// Start the game loop
	while (window.isOpen())
	{
		// Process events
		sf::Event event;
		while (window.pollEvent(event)) {
			if (event.type == sf::Event::Closed) {
				window.close();
			} else if(event.type == sf::Event::KeyPressed) {
				// the user interface: SPACE pauses and plays, ESC quits
				switch (event.key.code) {
					case sf::Keyboard::Space:
						togglePlayPause(music);
						break;
					case sf::Keyboard::Escape:
						window.close();
						break;
					default:
						break;
				}
			}
		}
		// Clear screen
		window.clear();
		// Draw the sprite
		window.draw(sprite);
		// Update the window
		window.display();
	}

	return EXIT_SUCCESS;
}
```
## <a name="3" />Makefile [ [Top] ](#top)
```make
CC=g++
CFLAGS=-std=c++11 -g -O0

# IMPORTANT
# replace <TODO> with your installation directories
#
SFML=TODO
MPG123=<TODO>

INCLUDESFML=$(SFML)/include
INCLUDEMPG123=$(MPG123)/include

LIBSFML=$(SFML)/lib
LIBMPG123=$(MPG123)/lib

INCLUDES := -I$(INCLUDEMPG123) -I$(INCLUDESFML)
# for non-debug builds
LIBS     := -L$(LIBMPG123) -lmpg123 -L$(LIBSFML) -lsfml-graphics -lsfml-window -lsfml-system -lsfml-audio
# for debug builds
#LIBS     := -L$(LIBMPG123) -lmpg123 -L$(LIBSFML) -lsfml-graphics-d -lsfml-window-d -lsfml-system-d -lsfml-audio-d

AUDIOEXE := audioexe

all: $(AUDIOEXE)

$(AUDIOEXE): Main.o
	$(CC) $(CFLAGS) $< -o $@ SoundFileReaderMp3.o $(LIBS)

Main.o: Main.cpp SoundFileReaderMp3.o
	$(CC) $(INCLUDES) $(CFLAGS) -c $< -o $@

SoundFileReaderMp3.o: SoundFileReaderMp3.cpp SoundFileReaderMp3.hpp
	$(CC) $(INCLUDES) $(CFLAGS) -c $< -o $@
 
clean:
	rm $(AUDIOEXE) *.o
```

## <a name="4" />Bonus: A possible seek implementation (by tschumacher) [ [Top] ](#top)
```cpp
void SoundFileReaderMp3::seek(sf::Uint64 sampleOffset) {
	// tschumacher: Based on this example: https://github.com/gypified/libmpg123/blob/master/doc/examples/feedseek.c
	// tschumacher: Spent a lot of time to figure out how to make that working with SFML
	// tschumacher: This implementation does not protect you from going out of bounds (seeking beyond music duration), seek() itself does never fail, but when you go out of bounds, the read() will fail.

	// Helper variables
	sf::Int64 read; // How many units were read?
	int status; // mpg123 status
	off_t inputOffset; // Input offset from mpg123
	off_t seekOffset; // Seek offset from SFML

	// Calculate seek offset from sfml offset by removing the channel count
	// tschumacher: Important, SFML multiplies the channel count in
	seekOffset = static_cast<off_t>(sampleOffset) / m_outputFormat.m_channels; // tschumacher: You need to make m_channels accessible

	// Feed seek to sample offset
	while ((status = mpg123_feedseek(handle_, seekOffset, SEEK_SET, &inputOffset)) == MPG123_NEED_MORE)
	{

		// Read one unit
		read = m_stream->read(buffer_, sizeof(unsigned char));

		// Nothing to read
		if (read <= 0)
			break;

		// Try to feed buffer into mpg123
		if (mpg123_feed(handle_, buffer_, static_cast<size_t>(read)) == MPG123_ERR)
		{
			std::cerr << "seek() @mpg123_feed(): " << mpg123_strerror(handle_) << std::endl;
			return;
		}
	}

	// Feedseek failed
	if (status == MPG123_ERR)
	{
		std::cerr << "seek() @mpg123_feedseek(): " << mpg123_strerror(handle_) << std::endl;
		return;
	}

	// Set correct position in stream
	m_stream->seek(inputOffset);
}
```