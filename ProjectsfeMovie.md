# sfeMovie project

sfeMovie is a small C++ library that allows you playing movies in SFML based applications. It relies on SFML for the rendering process and FFmpeg for the decoding process. It supports both audio and video, and basic controls. This project has been written to work closely with SFML and tries to keep the same easy-to-use paradigm,
while remaining coherent with SFML's conventions.

<!-- <a href="http://lucas.soltic.perso.esil.univmed.fr/img/sfemovie.png"><img src="http://lucas.soltic.perso.esil.univmed.fr/img/sfemovie.png" width="616" height="491"
title="Aperçu du film Sintel" alt="Aperçu du film Sintel"/></a> -->

###Quick links###

1. [How to use sfeMovie?](#howto)
2. [License](#license)
3. [System requirements](#requirements)
4. [Dependencies](#dependencies)
5. [Download links](#downloads)
6. [Supported file formats](#formats)
7. [Features status](#features)
8. [Contact](#contact)
9. [Thanks](#thanks)


## <a name="howto" />How to use sfeMovie ? [ [Top] ](#top)

Here is a sample code showing how to use sfeMovie, the setup instructions follow.

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

Make sure you downloaded sfeMovie (see [Download links](#downloads)) and have all of the headers and libraries ready to use.
I'm not going to explain how to use a library, most of it is like for any other library. I'll just focus on some specific points.

###General notes###

- Make sure you correctly set the headers and libraries search path to find both sfeMovie's and SFML's headers and libraries
- Link your product against sfeMovie (libsfe-movie) and SFML 2.0

###Mac OS X specific###

libsfe-movie has been built in a way that allows you to put the dynamic library in your bundle application, for distribution purpose. The copying process can be done manually or automatically through Xcode. If you want to do it manually, make sure to copy the library to your_app.app/Contents/Frameworks. Create the Frameworks directory if needed.

When using Xcode, you can automatize this process by [creating a Copy File build phase](http://developer.apple.com/library/mac/#recipes/xcode_help-project_editor/Articles/CreatingaCopyFilesBuildPhase.html) and setting the Destination of this phase to "Frameworks". Then add libsfe-movie to this phase, this will ask Xcode to copy the library to your application at build time.


## <a name="license" />License [ [Top] ](#top)

libsfe-movie is statically linked against FFmpeg. Thus you don't need to care about
the FFmpeg libraries, but therefore **sfeMovie is itself covered by the LGPL license**.

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
There is no binary available for now.

However you can view and download the latest sources from the [official Git repository](https://github.com/Yalir/sfeMovie). To build sfeMovie, make sure you have cmake, make and gcc installed (for Windows, make sure these are in the PATH environment variable). Then run build.sh from a command line interpreter (see [MSYS](http://www.mingw.org/wiki/msys) for Windows).

## <a name="formats" />Supported file formats [ [Top] ](#top)
The supported file formats depend on the FFmpeg configuration
flags used at compilation time.
The FFmpeg library provided with sfeMovie has been built with the default flags and supports the following formats:
###Short list of audio formats###
AAC, AC3, FLAC, MP3, PCM, Vorbis, WMA

###Short list of video formats###
FLV, H264, MPEG4, Theora, VP6, WMV

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

## <a name="features" />Features status [ [Top] ](#top)
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
I (Lucas Soltic) would like to thank some people for their great help!

- Laurent Gomilla for his excellent work on SFML
- Loïc Siquet for his general support and work
- Alexandre Janniaux for his help on the Linux port
- Martin Bohme for his FFmpeg tutorial (<http://dranger.com/ffmpeg/>)
- Xorlium and timidouveg for their video integration works from which was partly inspired sfeMovie (see <http://www.sfml-dev.org/wiki/en/sources/video_integration> and <http://www.sfml-dev.org/wiki/fr/tutoriels/integrervideo>)
- and all of the other people who helped me in any way :) 