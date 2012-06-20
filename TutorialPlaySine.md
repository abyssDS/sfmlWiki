# Playing a Sine Wave
This is intended for newcomers to SFML who want to experiment a bit with the audio library in 2.0. The way sound is loaded in SFML is via the `SoundBuffer` class. We give `loadFromSamples()` an array of 16 bit integers to load up along with how many samples there are in the array, how many channels we're dealing with (we're only going to be working with mono in this tutorial) and the sample rate of the audio. Sample rate just tells the computer exactly how many samples it should be processing in one second. We're going to use a standard sample rate of 44100 for this and we're going to load up exactly one second of audio. This means that the number of samples will have to match the sample rate. Once we load up the `SoundBuffer`, we just create a `Sound` class and point it to the buffer we just created with `setBuffer()` and set it to loop with `setLoop()` so that it continuously plays. After this, the audio is playing but if the program ends, the audio will end, so we put in a while loop that runs infinitely without taking up too many resources by forcing the program to sleep every 100 milliseconds. Here's the code just for playing an `Int16` array, but there's no real data in it yet.

```cpp
#include <SFML/Audio.hpp>
#include <iostream>

int main() {
	const unsigned SAMPLES = 44100;
	const unsigned SAMPLE_RATE = 44100;
	
	sf::Int16 raw[SAMPLES];
	
	sf::SoundBuffer Buffer;
	if (!Buffer.loadFromSamples(raw, SAMPLES, 1, SAMPLE_RATE)) {
		std::cerr << "Loading failed!" << std::endl;
		return 1;
	}

	sf::Sound Sound;
	Sound.setBuffer(Buffer);
	Sound.setLoop(true);
	Sound.play();
	while (1) {
		sf::sleep(sf::milliseconds(100));
	}
	return 0;
}
```

Now here comes the fun part. We want to load up `raw` with a sine wave that plays at 440hz (A4). We know that specifying the frequency of a sine wave is a simple as `sin(x*2π*frequency)`, but we have to remember that the sample rate dictates our time domain. So we have to multiply by increments of `440/44100` in order to get a sine wave that properly fits our sample rate. The last part is the amplitude. We will pick a nice and loud amplitude so you can hear it clearly over your speakers. 30,000 should do.

```cpp
const unsigned AMPLITUDE = 30000;
const double TWO_PI = 6.28318;
const double increment = 440./44100;
double x = 0;
for (unsigned i = 0; i < SAMPLES; i++) {
	raw[i] = AMPLITUDE * sin(x*TWO_PI);
	x += increment;
}
```
And that's it! Run the code and see if you can hear it. Remember to compile with `-lsfml-audio`. Here is the final code if you just want to copy & paste it:

```cpp
#include <SFML/Audio.hpp>
#include <cmath>
#include <iostream>

int main() {
	const unsigned SAMPLES = 44100;
	const unsigned SAMPLE_RATE = 44100;
	const unsigned AMPLITUDE = 30000;
	
	sf::Int16 raw[SAMPLES];

	const double TWO_PI = 6.28318;
	const double increment = 440./44100;
	double x = 0;
	for (unsigned i = 0; i < SAMPLES; i++) {
		raw[i] = AMPLITUDE * sin(x*TWO_PI);
		x += increment;
	}
	
	sf::SoundBuffer Buffer;
	if (!Buffer.loadFromSamples(raw, SAMPLES, 1, SAMPLE_RATE)) {
		std::cerr << "Loading failed!" << std::endl;
		return 1;
	}

	sf::Sound Sound;
	Sound.setBuffer(Buffer);
	Sound.setLoop(true);
	Sound.play();
	while (1) {
		sf::sleep(sf::milliseconds(100));
	}
	return 0;
}
```

Now try to get this playing the US dial tone. Here's a hint: it's made up of the frequencies 350hz and 440hz. Sine waves are added when combined.