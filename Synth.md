# Synthesizer
## Introduction
Synthesizer is a simple tool to enable creating sound effect programatically. It does not replace advanced sound creation tools and is therefore most appropriate for retro style games and sound effects. It uses standard SFML Sound and SoundBuffer classes for manipulating sounds later in the code, just generation is done with code and not by uploading sound files.

Inspired by the source code of the Sound Synthesizer tutorial series by One Lone Coder http://onelonecoder.com/

## Basic Features
It is capable generating six different sound waves:
* Sine
* Square
* Triangle
* Analogue Saw
* Digital Saw
* White noise

In addition to clean waves it allows stacking different types of waves on each other to generate more interesting and complex sound effects.

[Image]

## Usage
Library is contained in single Synth.hpp file and only needs to be included where we generate sounds. Sounds effects are defined in two structs and one function call:

Envelope is defined using struct with following parameters
* Attack time in seconds
* Decay time in seconds
* Sustain time in seconds
* Release time in seconds
* Start amplitude 0.0 - 1.0
* Sustain amplitude 0.0 - 1.0

Tones are defined using struct with following parameters
* Sound Wave type
* Start frequency in Hz
* End frequency in Hz
* Amplitude 0.0 - 1.0

Calling sound generation function
Simple version with one tone:
```
Synth::generate(
    &soundBuffer,    // address of sound buffer
    env,             // envelope struct
    tone,            // tone struct
    20000,           // Master Volume
    44100);          // Sample rate
```

## Examples
Usage example for simple sound effect e.g. Laser zap:
```
Synth::sEnvelope envelope = { 0.0, 0.0, 0.0, 0.3, 1.0, 1.0 };
Synth::sTone tone = { Synth::OSC_SAW_DIG, 900.0, 500.0, 0.8 };
Synth::generate(&soundBuffer, envelope, tone, 20000, 44100);
```

For more complex sounds we create several tone struct definitions and push them in vector and call generate like so:
```
// Harmonica
Synth::sEnvelope env = { 0.05, 1.0, 2.0, 0.1, 1.0, 0.95 };
Synth::sTone base = { Synth::OSC_SQUARE, 440.0, 440.0, 0.65 };         // A4
Synth::sTone harmonics = { Synth::OSC_SQUARE, 880.0, 880.0, 0.3 };     // A5
Synth::sTone noise = { Synth::OSC_NOISE, 880.0, 880.0, 0.05 };         // a bit of noise

std::vector <Synth::sTone> tones;
tone.push_back(base);
tone.push_back(harmonics);
tone.push_back(noise);

Synth::generate(&soundBuffer, env, tones, 20000, 44100);
```
## Synth.hpp
```
#pragma once
#ifndef SYNTH_HPP
#define SYNTH_HPP

////////////////////////////////////////////////////////////
// Headers
////////////////////////////////////////////////////////////
#include <SFML/Audio.hpp>
#include <cmath>

namespace Synth
{
const enum eWaveType { OSC_SINE, OSC_SQUARE, OSC_TRIANGLE, OSC_SAW_ANA, OSC_SAW_DIG, OSC_NOISE };

struct sEnvelope
{
    double dAttackTime = 0.0;
    double dDecayTime = 0.0;
    double dSustainTime = 1.0;
    double dReleaseTime = 0.0;
    double dStartAmplitude = 1.0;
    double dSustainAmplitude = 1.0;
};
struct sTone
{
    eWaveType waveType = OSC_SINE;
    double dStartFrequency = 440.0;
    double dEndFrequency = 440.0;
    double dAmplitude = 1.0;
};
    
////////////////////////////////////////////////////////////
// Converts frequency (Hz) to angular velocity
////////////////////////////////////////////////////////////
const double PI = 3.14159265359;
double w(const double dHertz)
{
    return dHertz * 2.0 * PI;
}


////////////////////////////////////////////////////////////
// Multi-Function Oscillator
//
// This function was mostly "borrowed" from One Lone Coder blog at
// wwww.onelonecoder.com
//
////////////////////////////////////////////////////////////
double osc(double dHertz, double dTime, eWaveType waveType)
{
    switch (waveType)
    {
    case OSC_SINE: // Sine wave bewteen -1 and +1
        return sin(w(dHertz) * dTime);

    case OSC_SQUARE: // Square wave between -1 and +1
        return sin(w(dHertz) * dTime) > 0 ? 1.0 : -1.0;

    case OSC_TRIANGLE: // Triangle wave between -1 and +1
        return asin(sin(w(dHertz) * dTime)) * (2.0 / PI);

    case OSC_SAW_ANA: // Saw wave (analogue / warm / slow)
    {
        double dOutput = 0.0;
        // this variable defines warmth, larger the number harsher and more similar do OSC_SAW_DIG sound becomes
        double dWarmth = 30.0;

        for (double n = 1.0; n < dWarmth; n++)
            dOutput += (sin(n * w(dHertz) * dTime)) / n;

        return dOutput * (2.0 / PI);
    }

    case OSC_SAW_DIG: // Saw Wave (optimised / harsh / fast)
        return (2.0 / PI) * (dHertz * PI * fmod(dTime, 1.0 / dHertz) - (PI / 2.0));

    case OSC_NOISE: // Pseudorandom noise
        return 2.0 * ((double)rand() / (double)RAND_MAX) - 1.0;

    default:
        return 0.0;
    }
}

////////////////////////////////////////////////////////////
// Amplitude Modulator
////////////////////////////////////////////////////////////
double amplitude(double dTime, sEnvelope env)
{
    double dTimeOn = 0;
    double dTimeOff = env.dAttackTime + env.dDecayTime + env.dSustainTime;
    double dAmplitude = 0.0;

    if (dTime > dTimeOff)                                                                                                                  // Release phase
        dAmplitude = ((dTime - dTimeOff) / env.dReleaseTime) * (0.0 - env.dSustainAmplitude) + env.dSustainAmplitude;

    else if (dTime > (env.dAttackTime + env.dDecayTime))                                                                                   // Sustain phase
        dAmplitude = env.dSustainAmplitude;

    else if (dTime > env.dAttackTime && dTime <= (env.dAttackTime + env.dDecayTime))                                                       // Decay phase
        dAmplitude = ((dTime - env.dAttackTime) / env.dDecayTime) * (env.dSustainAmplitude - env.dStartAmplitude) + env.dStartAmplitude;

    else if (dTime <= env.dAttackTime)                                                                                                     // Attack phase
        dAmplitude = (dTime / env.dAttackTime) * env.dStartAmplitude;

    // Amplitude should not be negative, check just in case
    if (dAmplitude <= 0.000)
        dAmplitude = 0.0;

    return dAmplitude;
}



////////////////////////////////////////////////////////////
/// \brief Generate sound and store in SoundBuffer
///
/// This function uses case-in
///
/// \param buffer is address to SoundBuffer where the result will be stored
/// \param envelope structure defining the ADSR Envelope
/// \param tones vector of tone structures to be stacked
/// \param master volume for the volume of sound
/// \param sample rate to set quality of sound
///
/// \return True if the sound was generate, false if it failed
///
////////////////////////////////////////////////////////////
bool generate(sf::SoundBuffer* buffer, sEnvelope env, std::vector<sTone> tones, unsigned uMasterVol, unsigned uSampleRate)
{
    if (!buffer)
        return false;

    // Calculate and allocate buffer
    double dTotalDuration = env.dAttackTime + env.dDecayTime + env.dSustainTime + env.dReleaseTime;
    unsigned iBufferSize = unsigned(dTotalDuration * uSampleRate);
    sf::Int16 * iRaw;
    iRaw = new sf::Int16[iBufferSize];

    // Generate sound
    double dIncrement = 1.0 / double(uSampleRate);
    double dTime = 0.0;
    for (unsigned i = 0; i < iBufferSize; i++)
    {
        double signal = 0.0;
            
        // Generate multiple tones and stack them together
        for (sTone t : tones)
        {
            double dFrequency = t.dStartFrequency + ((t.dEndFrequency - t.dStartFrequency) * (double(i) / double(iBufferSize)));
            signal = signal + (osc(dFrequency, dTime, t.waveType) * t.dAmplitude);
        }

        // Calculate Amplitude based on ADSR envelope
        double dEnvelopeAmplitude = amplitude(dTime, env);

        // Apply master volume, envelope and store to buffer
        *(iRaw + i) = sf::Int16(uMasterVol * signal * dEnvelopeAmplitude);

        dTime += dIncrement;
    }

	// Load into SFML SoundBuffer
    bool bSuccess = buffer->loadFromSamples(iRaw, iBufferSize, 1, uSampleRate);
    delete[] iRaw;

    return bSuccess;
}

////////////////////////////////////////////////////////////
/// \brief Generate sound and store in SoundBuffer
///
/// This function uses case-in
///
/// \param buffer is address to SoundBuffer where the result will be stored
/// \param envelope structure defining the ADSR Envelope
/// \param tone structure for simple tone definition
/// \param master volume for the volume of sound
/// \param sample rate to set quality of sound
///
/// \return True if the sound was generate, false if it failed
///
////////////////////////////////////////////////////////////
bool generate(sf::SoundBuffer* buffer, sEnvelope env, sTone tone, unsigned uMasterVol, unsigned uSampleRate)
{
    std::vector<sTone> tones;
    tones.push_back(tone);
    return generate(buffer, env, tones, uMasterVol, uSampleRate);
}
}
#endif  // SYNTH_HPP
```
## Sound.cpp
```
#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include "Synth.hpp"

int main()
{
    sf::RenderWindow window(sf::VideoMode(200, 200), "Synthesizer Demo!");

    sf::SoundBuffer soundBuffer;
    sf::Sound demoSound;
    
    // LASER
    Synth::sEnvelope envelope = { 0.0, 0.0, 0.0, 0.3, 1.0, 1.0 };
    Synth::sTone tone = { Synth::OSC_SAW_DIG, 900.0, 500.0, 0.8 };
    Synth::generate(&soundBuffer, envelope, tone, 20000, 44100);
    demoSound.setBuffer(soundBuffer);

    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                window.close();
            if (event.type == sf::Event::KeyPressed)
                demoSound.play();
        }
    }

    return 0;
}
```

## Sounds Library
