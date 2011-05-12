# sfeMovie project

sfeMovie is a small C++ library that lets you play movies in SFML based applications. It relies on SFML for the rendering process and FFmpeg for the decoding process. It supports both audio and video, and basic controls. This project has been written to work closely with SFML and tries to keep the same easy-to-use paradigm,
while remaining coherent with SFML's conventions.

<!-- <a href="http://lucas.soltic.perso.esil.univmed.fr/img/sfemovie.png"><img src="http://lucas.soltic.perso.esil.univmed.fr/img/sfemovie.png" width="616" height="491"
title="Aperçu du film Sintel" alt="Aperçu du film Sintel"/></a> -->

###Quick links###

1. [How to use sfeMovie?](#howto)
2. [License and legal notes](#license)
3. [System requirements](#requirements)
4. [Dependencies](#dependencies)
5. [Download links](#downloads)
6. [Building sfeMovie](#build)
7. [Supported file formats](#formats)
8. [Features status](#features)
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

libsfe-movie has been built in a way that allows you to put the dynamic library in your bundle application, for distribution purpose. The copying process can be done manually, or automatically through Xcode. If you want to do it manually, make sure the library is copied to your_app.app/Contents/Frameworks. Create the Frameworks directory if needed.

When using Xcode, you can automatize this process by [creating a Copy File build phase](http://developer.apple.com/library/mac/#recipes/xcode_help-project_editor/Articles/CreatingaCopyFilesBuildPhase.html) and setting the Destination of this phase to "Frameworks". Then add libsfe-movie to this phase, this will ask Xcode to copy the library into your application at build time.


## <a name="license" />License [ [Top] ](#top)

libsfe-movie is statically linked against FFmpeg, that is licensed under [LGPL 2.1](http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html). Thus you don't need to care about the FFmpeg libraries, but **sfeMovie is itself covered by the LGPL v2.1 license**. As for the legal side, note that you can download the sfeMovie sources from the [official Git repository](https://github.com/Yalir/sfeMovie).

Basically, this means you can use sfeMovie for ANY project without ANY restriction until
libsfe-movie is linked dynamically to your software. As soon as you link libsfe-movie statically,
your software is also covered by the LGPL licence and you MUST provide the sources of your
application/library. This is the main restriction.

For more information on LGPL, have a look at <http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html>.

###Legal notes###
As you may know, patents on video codecs is a complex issue. FFmpeg is distributed under LGPL and does not provide any binary version of the library. This is to avoid legal issue : decoders' source code can be freely distributed, but not binaries (though it still depends on whether the patent holder is strict, because FFmpeg is not an end-software product). Moreover patents do not apply in some countries. That's why [VideoLAN doesn't pay any fee to distribute VLC](http://www.videolan.org/legal.html). But as I'm not a specialist and I don't know the details about laws, the binaries I'm providing only support the free decoders : flac, vorbis, theora. If you want support for more decoders, you'll have to build sfeMovie yourself. A script is provided so that the whole compilation process can be performed flawlessly. Note that you're given the ability to exactly choose which decoders you want to enable in the final sfeMovie binary.

You don't have to worry about licenses and royalties if you're a lonely developer working on a private project. But if you plan to distribute your product to "a lot" of people, or as a commercial purpose, you should be careful.

Considering this, you're responsible for checking the possible fees and licenses you may need, depending on your country, needs and situation. Here is a little (non official) sum up of the different fees for the main decoders. The information I'm showing here is the one I got from the different patents holders.

<table>
  <tr>
    <th>Video codec</th>
    <th>Royalties</th>
    <th>Source</th>
  </tr>

  <tr>
    <td>H.264 (MPEG4 AVC)</td>
    <td>free under 100 000 units distributed per year (license still required), then US $0.20 per unit after first 100 000 units per year</td>
    <td><a href="http://www.mpegla.com/main/programs/AVC/Documents/AVC_TermsSummary.pdf">MPEG LA (PDF)</a></td>
  </tr>

  <tr>
    <td>MPEG4</td>
    <td>free under 50 000 units distributed per year (license still required), then US $0.25 per unit after first 50 000 units per year</td>
    <td><a href="http://www.mpegla.com/main/programs/M4V/Documents/m4vweb.ppt">MPEG LA (PowerPoint)</a></td>
  </tr>

  <tr>
    <td>Theora</td>
    <td>free for any use</td>
    <td><a href="http://www.theora.org/faq/">Theora</a></td>
  </tr>

  <tr>
    <td>VP6</td>
    <td>?</td>
    <td>?</td>
  </tr>

  <tr>
    <td>WMV</td>
    <td><!-- free for any use? -->?</td>
    <td><!-- <a href="http://www.microsoft.com/windows/windowsmedia/licensing/LicensingAVServices.aspx">Microsoft's website?</a> -->?</td>
  </tr>

</table>

<table>
  <tr>
    <th>Audio codec</th>
    <th>Royalties</th>
    <th>Source</th>
  </tr>

  <tr>
    <td>AAC</td>
    <td>US$ 0.48 per unit with a maximum annual payment per product of US$ 32 000</td>
    <td><a href="http://www.vialicensing.com/licensing/aac-fees.aspx">Via Licensing</a></td>
  </tr>
  
  <tr>
    <td>AC3</td>
    <td>?</td>
    <td>?</td>
  </tr>
  
  <tr>
    <td>FLAC</td>
    <td>free for any use</td>
    <td><a href="http://flac.sourceforge.net/license.html">flac</a></td>
  </tr>
  
  <tr>
    <td>MP3</td>
    <td><b>maximum</b> between US$ 15 000 and US$ 0.75 per unit, or US$ 2 500 per title for games</td>
    <td><a href="http://www.mp3licensing.com/royalty/">mp3licensing.com</a></td>
  </tr>
  
  <tr>
    <td>PCM</td>
    <td>?</td>
    <td>?</td>
  </tr>
  
  <tr>
    <td>Vorbis</td>
    <td>free for any use</td>
    <td><a href="http://www.vorbis.com/faq/#flic">Vorbis</a></td>
  </tr>
  
  <tr>
    <td>WMA</td>
    <td>?</td>
    <td>?</td>
  </tr>

</table>

## <a name="requirements" />System requirements [ [Top] ](#top)
sfeMovie is known to work fine on both Mac OS X and Windows, but is unstable on Linux.

###Linux###
- OS version: N/A
- Architecture: N/A

###Mac OS X###
- OS version: Mac OS X 10.5 and later
- Architecture: Intel 32 bits

###Windows###
- OS version: Windows XP and later
- Architecture: Intel 32 bits

## <a name="dependencies" />Dependencies [ [Top] ](#top)
sfeMovie relies on SFML 2.0, sndfile and OpenAL, you can find the required libraries in the "deps" (dependencies) directory. The SFML libraries were built with the sources of the official Git repository on 2011-05-11.

There is no other specific dependency you have to care about.

## <a name="downloads" />Download links [ [Top] ](#top)

<b>Important note:</b> the given binaries only provide support for the free decoders (flac, vorbis and theora). If you want to use other decoders, please read the [License section](#license) (and especially the legal notes), and consider [building sfeMovie yourself](#build).
<table>
	<tr>
		<th colspan=3>sfeMovie beta_20110512</th>
	</tr>
	<tr>
		<th>Operating System</th>
		<th>Binaries</th>
		<th>Sources</th>
	</tr>
	
	<tr>
		<td>Linux</td>
		<td>N/A</td>
		<td rowspan=3><a href="https://github.com/downloads/Yalir/sfeMovie/sfeMovie-src-beta_20110512.zip">sfeMovie-src-beta_20110512.zip</a></td>
	</tr>
	
	<tr>
		<td>Mac OS X</td>
		<td><a href="https://github.com/downloads/Yalir/sfeMovie/sfeMovie-macosx-beta_20110512.zip">sfeMovie-macosx-beta_20110512.zip</a></td>
	</tr>
	
	<tr>
		<td>Windows</td>
		<td><a href="https://github.com/downloads/Yalir/sfeMovie/sfeMovie-windows-beta_20110512.zip">sfeMovie-windows-beta_20110512.zip</a></td>
	</tr>
</table>
<table>
	<tr>
		<th colspan=3>Latest sfeMovie files (from the Git repository)</th>
	</tr>
	<tr>
		<th>Operating System</th>
		<th>Sources</th>
	</tr>
	
	<tr>
		<td>Linux</td>
		<td rowspan=3><a href="https://github.com/Yalir/sfeMovie/zipball/master">zip archive</a></td>
	</tr>
	
	<tr>
		<td>Mac OS X</td>
	</tr>
	
	<tr>
		<td>Windows</td>
	</tr>
</table>
 

## <a name="build" />Building sfeMovie [ [Top] ](#top)



To build sfeMovie, make sure you have CMake, Make and GCC installed (for Windows, make sure these are in the PATH environment variable), and download the latest sources (see [download links](#downloads)). GCC is the only supported compiler. Then run build.sh from a command line interpreter (see [MinGW](http://sourceforge.net/projects/mingw/files/Automated%20MinGW%20Installer/mingw-get-inst/) for Windows).


## <a name="formats" />Supported file formats [ [Top] ](#top)
The supported file formats depend on the FFmpeg configuration
flags used at compilation time. When building FFmpeg, you can choose which decoders you want sfeMovie to support. Note that the provided sfeMovie binaries only support the free decoders : flac, vorbis and theora. Building FFmpeg with the default configuration flags enables the following formats:
###Short list of audio formats###
AAC, AC3, FLAC, MP3, PCM, Vorbis, WMA

###Short list of video formats###
H.264 (aka MPEG-4 AVC), MPEG4, Theora, VP6, WMV

Note that the FLV format is a container for H.264 and VP6, and that FFmpeg supports this container.

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

## <a name="thanks" />Thanks [ [Top] ](#top)
I (Lucas Soltic) would like to thank some people for their great help!

- Laurent Gomilla for his excellent work on SFML
- Loïc Siquet for his general support and work
- Alexandre Janniaux for his help on the Linux port
- Martin Bohme for his FFmpeg tutorial (<http://dranger.com/ffmpeg/>)
- Xorlium and timidouveg for their video integration works from which was partly inspired sfeMovie (see <http://www.sfml-dev.org/wiki/en/sources/video_integration> and <http://www.sfml-dev.org/wiki/fr/tutoriels/integrervideo>)
- and all of the other people who helped me in any way :) 

If you ever wished to contact me, you can [drop me a line through the SFML's private messaging system](http://www.sfml-dev.org/forum/privmsg.php?mode=post&u=395) or post on the [sfeMovie's forum topic](http://www.sfml-dev.org/forum/viewtopic.php?t=3463).