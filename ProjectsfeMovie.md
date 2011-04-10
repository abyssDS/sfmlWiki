# sfeMovie project

sfeMovie is a small C++ library that allows you playing movies in SFML based applications. It relies on SFML for the rendering process and FFmpeg for the decoding process. It supports both audio and video, and basic controls. This project has been written to work closely with SFML and tries to keep the same easy-to-use paradigm,
while being coherent with SFML's conventions.

###Quick links###

1. [How to use sfeMovie?](#howto)
2. [License](#license)
3. [System requirements](#requirements)
4. [Dependencies](#dependencies)
5. [Download links](#downloads)
6. [Supported file formats](#formats)
7. [Current status about features](#status)
8. [Contact](#contact)
9. [Thanks](#thanks)


## <a name="howto" />How to use sfeMovie ? [ [Top] ](#top)

Here is a sample code showing how to use sfeMovie, the instructions for each OS follow.

```cpp
#include <SFML/Graphics.hpp>
#include <Movie.h>

int main()
{
    sf::RenderWindow window(sf::VideoMode(640, 480), "SFE Movie Player");
    sfe::Movie movie;

    if (!movie.OpenFromFile("movie.avi"))
        return 1;

    movie.ResizeToFrame(0, 0, 640, 480);
    movie.Play();

    while (window.IsOpened())
    {
        sf::Event ev;
        while (window.GetEvent(ev))
        {
            if (ev.Type == sf::Event::Closed)
                window.Close();
        }

        window.Clear();
        window.Draw(movie);
        window.Display();
    }

    return 0;
}
```


###Linux###
N/A

###Mac OS X###


**With Xcode**

- Add the directory where you put Movie.h to the headers search path.
- Drag every library from "lib" and "extlib" to your project.
- Add a Copy File build phase to your project, and set the "Destination" of this phase to "Frameworks".
- Add all of the imported libraries to this build phase.
- Write your code and build your app.

**Without Xcode**

- Write your code.
- Add the directory where you put Movie.h to the header search path (through option -I).
- Add the directory/ies containing the libraries originally found in "lib" and "extlib" to the library search path (through option -L)
- Compile your application.
- Copy the libraries to your_app.app/Contents/Frameworks

###Windows###
N/A

## <a name="license" />License [ [Top] ](#top)

libsfe-movie is statically linked against FFmpeg. Thus you don't need to care about
the FFmpeg libraries, but **libsfe-movie is itself covered by the LGPL license**.

Basically, this means you can use sfeMovie for ANY project without ANY restriction until
libsfe-movie is linked dynamically to your software. As soon as you link libsfe-movie statically,
your software is also covered by the LGPL licence and you MUST provide the sources of your
application/library. 

See <http://www.gnu.org/licenses/lgpl.html> for more information on LGPL.

## <a name="requirements" />System requirements [ [Top] ](#top)
sfeMovie is known to work fine on both Mac OS X and Windows, but is unstable on Linux.

###Linux###
- OS version: N/A
- Architecture: N/A

###Mac OS X###
- OS version: Mac OS X 10.5 and later
- Architecture: Intel 32 bits and Intel 64 bits

###Windows###
- OS version: Windows XP and later
- Architecture: Intel 32 bits

## <a name="dependencies" />Dependencies [ [Top] ](#top)
sfeMovie relies on SFML 2.0, you can find the required SFML libraries in the extlib directory. These were built with the sources of the official Git repository on $date$.

There is no other specific dependency you have to care about.

## <a name="downloads" />Download links [ [Top] ](#top)
N/A

## <a name="formats" />Supported file formats [ [Top] ](#top)
The supported file formats depend on the FFmpeg configuration
flags used at compilation time.
The FFmpeg library provided with sfeMovie has been built with the default flags and supports the following formats:
###Short list of audio formats###
AAC, AC3, FLAC, MP3, PCM, Vorbis, WMA

###Short list of video formats###
FLV, H264, MPEG4, Theora, WMV

###Comprehensive list including both audio and video formats###
aac, aasc, ac3, adpcm_4xm, adpcm_adx, adpcm_ct, adpcm_ea, adpcm_ea_maxis_xa,
adpcm_ea_r1, adpcm_ea_r2, adpcm_ea_r3, adpcm_ea_xas, adpcm_g726, adpcm_ima_amv,
adpcm_ima_dk3, adpcm_ima_dk4, adpcm_ima_ea_eacs, adpcm_ima_ea_sead,
adpcm_ima_iss, adpcm_ima_qt, adpcm_ima_smjpeg, adpcm_ima_wav, adpcm_ima_ws,
adpcm_ms, adpcm_sbpro_2, adpcm_sbpro_3, adpcm_sbpro_4, adpcm_swf, adpcm_thp,
adpcm_xa, adpcm_yamaha, alac, als, amrnb, amv, anm, ape, asv1, asv2, atrac1,
atrac3, aura, aura2, avs, bethsoftvid, bfi, bink, binkaudio_dct, binkaudio_rdft,
bmp, c93, cavs, cdgraphics, cinepak, cljr, cook, cscd, cyuv, dca, dnxhd, dpx,
dsicinaudio, dsicinvideo, dvbsub, dvdsub, dvvideo, dxa, eac3, eacmv, eamad,
eatgq, eatgv, eatqi, eightbps, eightsvx_exp, eightsvx_fib, escape124, ffv1,
ffvhuff, flac, flashsv, flic, flv, fourxm, fraps, frwu, gif, h261, h263, h263i,
h264, huffyuv, idcin, iff_byterun1, iff_ilbm, imc, indeo2, indeo3, indeo5,
interplay_dpcm, interplay_video, jpegls, kgv1, kmvc, loco, mace3, mace6, mdec,
mimic, mjpeg, mjpegb, mlp, mmvideo, motionpixels, mp1, mp2, mp3, mp3adu, mp3on4,
mpc7, mpc8, mpeg1video, mpeg2video, mpeg4, mpeg_xvmc, mpegvideo, msmpeg4v1,
msmpeg4v2, msmpeg4v3, msrle, msvideo1, mszh, nellymoser, nuv, pam, pbm, pcm_alaw,
pcm_bluray, pcm_dvd, pcm_f32be, pcm_f32le, pcm_f64be, pcm_f64le, pcm_mulaw,
pcm_s16be, pcm_s16le, pcm_s16le_planar, pcm_s24be, pcm_s24daud, pcm_s24le,
pcm_s32be, pcm_s32le, pcm_s8, pcm_u16be, pcm_u16le, pcm_u24be, pcm_u24le,
pcm_u32be, pcm_u32le, pcm_u8, pcm_zork, pcx, pgm, pgmyuv, pgssub, png, ppm,
ptx, qcelp, qdm2, qdraw, qpeg, qtrle, r210, ra_144, ra_288, rawvideo, rl2, roq,
roq_dpcm, rpza, rv10, rv20, rv30, rv40, sgi, shorten, sipr, smackaud, smacker,
smc, snow, sol_dpcm, sonic, sp5x, sunrast, svq1, svq3, targa, theora, thp,
tiertexseqvideo, tiff, tmv, truehd, truemotion1, truemotion2, truespeech, tscc,
tta, twinvq, txd, ulti, v210, v210x, vb, vc1, vcr1, vmdaudio, vmdvideo, vmnc,
vorbis, vp3, vp5, vp6, vp6a, vp6f, vqa, wavpack, wmapro, wmav1, wmav2, wmavoice,
wmv1, wmv2, wmv3, wnv1, ws_snd1, xan_dpcm, xan_wc3, xl, xsub, yop, zlib, zmbv

## <a name="status" />Current status about features [ [Top] ](#top)
###Currently implemented features###
- video output with any of the above video formats
- audio output with any of the above audio formats
- playing, pausing and stopping a movie
- getting the movie properties
- getting the current image
- scaling the movie drawable to fit a given frame, eventually keeping the original ratio

###Planned or missing features###
- Linux port
- seeking in the movie
- movie playback looping
- subtitles

## <a name="contact" />Contact [ [Top] ](#top)
## <a name="thanks" />Thanks [ [Top] ](#top)