---
title: TouchPlayer Documentation
author: error454
excerpt: The documentation for TouchPlayer
layout: post
permalink: /2011/10/16/touchplayer-documentation/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:1:{s:74:"http://mobilecoder.files.wordpress.com/2011/08/applications-multimedia.png";a:6:{s:8:"file_url";s:74:"http://mobilecoder.files.wordpress.com/2011/08/applications-multimedia.png";s:5:"width";s:3:"150";s:6:"height";s:3:"150";s:4:"type";s:5:"image";s:4:"area";s:5:"22500";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"1";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2011-10-17 00:10:46";}'
categories:
  - Homebrew
  - WebOS
tags:
  - avi
  - ffmpeg
  - flv
  - homebrew
  - mkv
  - mplayer
  - touchpad
  - touchplayer
  - vlc
  - webos
---
<table>
  <tr>
    <td>
      <img class="size-full wp-image-931 alignnone" style="border-color:initial;border-style:initial;border-width:0;" title="TouchPlayer - the video player for total badasses" src="{{ site.url }}/assets/uploads/2011/08/applications-multimedia.png" alt="" width="120" height="120" />
    </td>
    
    <td>
      <img src='' alt=''>
    </td>
  </tr>
</table>



# How to Get TouchPlayer

TouchPlayer is distributed through a private feed, you need to add it (mobilecoder.touchpadhp.info/mcfeed) so that TouchPlayer will show up in Preware.  Alternatively, if you know what you are doing, you can download the ipk directly (http://mobilecoder.touchpadhp.info/touchplayer/)

Directions for Preware

1.  Open Preware
2.  Tap the top left of the Preware screen and select **Manage Feeds**
3.  Scroll to the bottom and under New Feed, give the feed a name (any name) and enter/select the following corresponding details
1.  **URL**: mobilecoder.touchpadhp.info/mcfeed
2.  **IS Compressed**: NO

4.  Select **Add Feed**
5.  Again in Preware, select **Update feeds**
6.  Install (Search for TouchPlayer in Preware, select it and hit install
7.  Luna Restart (Preware menu, Luna Manager, Luna Restart)

# Features/What's New

## 1.0.7

*   Added UI field to paste rtsp streams
*   Added scale parameter to the play service call which basically does: args += -vf scale=: + inArgs.scale +  ;)
*   Playing a file now automatically kills existing mplayer instances

## 1.0.6a

*   Modified post-install script to attempt to fix install issues.  I was able to reproduce the issue on my device.  I verified that after fixing the files, a simple Luna restart did not resolve the issue but a reboot did.  Therefore, a reboot is now required after install

## 1.0.6

*   Added install check
*   Modified back-end launching, hopefully making emergency kill unecessary
*   Now uses TTF fonts, user can choose from fonts installed in /usr/share/fonts/. This makes old font.desc junk obsolete and should also immediately support all locals
*   Font scaling now works (sizes in the GUI are multipliers not points)
*   New API call for getting list of fonts
*   New API call for validating installation
*   New API parameters for playing videos
*   Updated to latest FFmpeg and mplayer as of 11/5/2011

## 1.0.5 (Experimental)

*   Improvements to back-end mplayer launching
*   Preliminary Cyrillic character support
*   Move subtitles into empty black area if possible

## 1.0.4

*   Runs mplayer underneath the hood meaning that most video formats are supported.
*   NEON accelerated YUV to RGB conversion
*   Pause/Fast forward/Rewind
*   Subtitle support

# Controls

<h2 style="padding-left:30px;">
  Pause
</h2>

<p style="padding-left:30px;">
  To pause the video, single tap on the screen.  You can also minimize the video window into card view.  To unpause, single tap again.
</p>

<h2 style="padding-left:30px;">
  Fast Forward/Rewind
</h2>

<p style="padding-left:30px;">
  To fast forward or rewind, swipe horizontally right or left.  1 finger = small increments, 2 fingers = large increments.
</p>

<h2 style="padding-left:30px;">
  Subtitles On/Off
</h2>

<p style="padding-left:30px;">
  To toggle subtitles on/off, swipe down with 3 fingers.  You will see a confirmation in the top/left of the video indicating the status of the subtitles.
</p>

<h2 style="padding-left:30px;">
  Subtitle Index Toggle
</h2>

<p style="padding-left:30px;">
  To cycle between the available subtitles, swipe up with 3 fingers.  You will see a confirmation in the top/left of the video indicating the name of the current set of subtitles.
</p>

# UI

<h2 style="padding-left:30px;">
  Font Size
</h2>

<p style="padding-left:30px;">
  The font size determines the size of the subtitles.
</p>

<h2 style="padding-left:30px;">
  Emergency Kill
</h2>

<p style="padding-left:30px;">
  As of 1.0.6, this should no longer be needed but I've left it in just in case.  Please report if you have threads that aren't being cleaned up.
</p>

<p style="padding-left:30px;">
  I apologize in advance that this button is even necessary.  I have noticed that sometimes mplayer does not clean up all of the threads that it spins up, occasionally this can cause issues launching a new instance of mplayer.  This button is here to resolve these issues.  Pressing this button will kill all running instances of mplayer.
</p>

<p style="padding-left:60px;">
  Q. When should I press this button?<br /></br> A. If suddenly you find that videos are not launching, press this button and then try launching your video again.
</p>

<p style="padding-left:60px;">
  Q. Can anything bad happen if I hit this button when I don't need to?<br /></br> A. Nothing bad will happen, feel free to press it without fear!
</p>

# Service Calls

For folks that would like to leverage TouchPlayer there are 3 service calls available:

## play

Plays a given file or stream.

### Sample service definition.

<pre>{
  name: playFile,
  kind: PalmService,
  service: palm://com.wordpress.mobilecoder.touchplayer.service,
  method: play,
  onSuccess: fileStarted
}
</pre>

### Arguments

source: string  The full path to file  
audio: boolean  Whether to enable audio or not  
font: string  The name of the font (for a list of names see below)  
<del>fontsize: integer</del>  
fontscale: integer  How large to scale the font  
<del>charset: en</del>  
movesubs: boolean  Whether to move subs into the black area  
onsuccess: function  A callback to be called once the play command completes  
scale: int  Adds the command  -vf scale=:*scale*

### Sample call

<pre>this.$.playFile.call(
{
    source: /media/internal/movie.mp4,
    audio: true,
    fontscale: 2,
    font: times,
    movesubs: false
});
</pre>

## killall

Kills all mplayer processes

### Sample service definition

<pre>{
  name: emergencyKill,
  kind: PalmService,
  service: palm://com.wordpress.mobilecoder.touchplayer.service,
  method: killall
}
</pre>

### Sample call

<pre>//Kill all instances of mplayer
this.$.emergencyKill.call();
</pre>

## getfonts

Returns a list of fonts stored in /usr/share/fonts

### Sample service definition

<pre>{
    name: getFontList,
    kind: enyo.PalmService,
    service: palm://com.wordpress.mobilecoder.touchplayer.service,
    method: getfonts,
    onSuccess: gotFonts
}
</pre>

### Sample success handler:

<pre>gotFonts: function(inSender, inResponse, inRequest){
   //Populate a font dialog box
   this.$.fontName.setItems(inResponse.reply);
}
</pre>

# Help!

For help, please post a comment below.  Note that comments are moderated and may not appear once you submit them.  I will receive an email when you leave a comment.

The <a href="http://wiki.multimedia.cx/index.php?title=MPlayer_FAQ" target="_blank">mplayer faq</a> may also be of interest.

The following is the output of the FFmpeg configuration step, this shows you which codecs are compiled in.

<pre>source path               .
C compiler                gcc
ARCH                      arm (generic)
big-endian                no
runtime cpu detection     no
ARMv5TE enabled           yes
ARMv6 enabled             yes
ARMv6T2 enabled           yes
ARM VFP enabled           yes
IWMMXT enabled            no
NEON enabled              yes
debug symbols             no
strip symbols             yes
optimize for size         no
optimizations             yes
static                    yes
shared                    no
postprocessing support    yes
new filter support        yes
network support           yes
threading support         pthreads
SDL support               yes
Sun medialib support      no
libdxva2 enabled          no
libva enabled             no
libvdpau enabled          no
AVISynth enabled          no
libcelt enabled           no
frei0r enabled            no
libcdio support           no
libdc1394 support         no
libdirac enabled          no
libfaac enabled           no
libaacplus enabled        no
libgsm enabled            no
libmodplug enabled        no
libmp3lame enabled        no
libnut enabled            no
libopencore-amrnb support no
libopencore-amrwb support no
libopencv support         no
libopenjpeg enabled       no
libpulse enabled          no
librtmp enabled           no
libschroedinger enabled   no
libspeex enabled          no
libstagefright-h264 enabled    no
libtheora enabled         no
libutvideo enabled        no
libvo-aacenc support      no
libvo-amrwbenc support    no
libvorbis enabled         no
libvpx enabled            no
libx264 enabled           no
libxavs enabled           no
libxvid enabled           no
openal enabled            no
zlib enabled              yes
bzlib enabled             no

Enabled decoders:
aac			dirac			mp1
aac_latm		dnxhd			mp1float
aasc			dpx			mp2
ac3			dsicinaudio		mp2float
adpcm_4xm		dsicinvideo		mp3
adpcm_adx		dvbsub			mp3adu
adpcm_ct		dvdsub			mp3adufloat
adpcm_ea		dvvideo			mp3float
adpcm_ea_maxis_xa	dxa			mp3on4
adpcm_ea_r1		eac3			mp3on4float
adpcm_ea_r2		eacmv			mpc7
adpcm_ea_r3		eamad			mpc8
adpcm_ea_xas		eatgq			mpeg1video
adpcm_g722		eatgv			mpeg2video
adpcm_g726		eatqi			mpeg4
adpcm_ima_amv		eightbps		mpegvideo
adpcm_ima_dk3		eightsvx_exp		msmpeg4v1
adpcm_ima_dk4		eightsvx_fib		msmpeg4v2
adpcm_ima_ea_eacs	eightsvx_raw		msmpeg4v3
adpcm_ima_ea_sead	escape124		msrle
adpcm_ima_iss		ffv1			msvideo1
adpcm_ima_qt		ffvhuff			mszh
adpcm_ima_smjpeg	flac			mxpeg
adpcm_ima_wav		flashsv			nellymoser
adpcm_ima_ws		flashsv2		nuv
adpcm_ms		flic			pam
adpcm_sbpro_2		flv			pbm
adpcm_sbpro_3		fourxm			pcm_alaw
adpcm_sbpro_4		fraps			pcm_bluray
adpcm_swf		frwu			pcm_dvd
adpcm_thp		g723_1			pcm_f32be
adpcm_xa		g729			pcm_f32le
adpcm_yamaha		gif			pcm_f64be
alac			gsm			pcm_f64le
als			gsm_ms			pcm_lxf
amrnb			h261			pcm_mulaw
amrwb			h263			pcm_s16be
amv			h263i			pcm_s16le
anm			h264			pcm_s16le_planar
ansi			huffyuv			pcm_s24be
ape			idcin			pcm_s24daud
ass			idf			pcm_s24le
asv1			iff_byterun1		pcm_s32be
asv2			iff_ilbm		pcm_s32le
atrac1			imc			pcm_s8
atrac3			indeo2			pcm_u16be
aura			indeo3			pcm_u16le
aura2			indeo5			pcm_u24be
avs			interplay_dpcm		pcm_u24le
bethsoftvid		interplay_video		pcm_u32be
bfi			jpeg2000		pcm_u32le
bink			jpegls			pcm_u8
binkaudio_dct		jv			pcm_zork
binkaudio_rdft		kgv1			pcx
bintext			kmvc			pgm
bmp			lagarith		pgmyuv
c93			loco			pgssub
cavs			mace3			pictor
cdgraphics		mace6			png
cinepak			mdec			ppm
cljr			mimic			prores
cook			mjpeg			prores_lgpl
cscd			mjpegb			ptx
cyuv			mlp			qcelp
dca			mmvideo			qdm2
dfa			motionpixels		qdraw
qpeg			svq1			vp3
qtrle			svq3			vp5
r10k			targa			vp6
r210			theora			vp6a
ra_144			thp			vp6f
ra_288			tiertexseqvideo		vp8
rawvideo		tiff			vqa
rl2			tmv			wavpack
roq			truehd			wmalossless
roq_dpcm		truemotion1		wmapro
rpza			truemotion2		wmav1
rv10			truespeech		wmav2
rv20			tscc			wmavoice
rv30			tta			wmv1
rv40			twinvq			wmv2
s302m			txd			wmv3
sgi			ulti			wmv3image
shorten			utvideo			wnv1
sipr			v210			ws_snd1
smackaud		v210x			xan_dpcm
smacker			vb			xan_wc3
smc			vc1			xan_wc4
snow			vc1image		xbin
sol_dpcm		vcr1			xl
sonic			vmdaudio		xsub
sp5x			vmdvideo		yop
srt			vmnc			zlib
sunrast			vorbis			zmbv

Enabled encoders:
a64multi		h263			pcm_u24le
a64multi5		h263p			pcm_u32be
aac			huffyuv			pcm_u32le
ac3			jpeg2000		pcm_u8
ac3_fixed		jpegls			pcx
adpcm_adx		ljpeg			pgm
adpcm_g722		mjpeg			pgmyuv
adpcm_g726		mp2			png
adpcm_ima_qt		mpeg1video		ppm
adpcm_ima_wav		mpeg2video		prores
adpcm_ms		mpeg4			qtrle
adpcm_swf		msmpeg4v2		ra_144
adpcm_yamaha		msmpeg4v3		rawvideo
alac			msvideo1		roq
amv			nellymoser		roq_dpcm
ass			pam			rv10
asv1			pbm			rv20
asv2			pcm_alaw		sgi
bmp			pcm_f32be		snow
dca			pcm_f32le		sonic
dnxhd			pcm_f64be		sonic_ls
dpx			pcm_f64le		srt
dvbsub			pcm_mulaw		svq1
dvdsub			pcm_s16be		targa
dvvideo			pcm_s16le		tiff
eac3			pcm_s24be		v210
ffv1			pcm_s24daud		vorbis
ffvhuff			pcm_s24le		wmav1
flac			pcm_s32be		wmav2
flashsv			pcm_s32le		wmv1
flashsv2		pcm_s8			wmv2
flv			pcm_u16be		xsub
g723_1			pcm_u16le		zlib
gif			pcm_u24be		zmbv
h261

Enabled hwaccels:

Enabled parsers:
aac			dvdsub			mpegaudio
aac_latm		flac			mpegvideo
ac3			h261			pnm
cavsvideo		h263			rv30
dca			h264			rv40
dirac			mjpeg			vc1
dnxhd			mlp			vp3
dvbsub			mpeg4video		vp8

Enabled demuxers:
aac			image2			pcm_u24le
ac3			image2pipe		pcm_u32be
act			ingenient		pcm_u32le
adf			ipmovie			pcm_u8
aea			iss			pmp
aiff			iv8			pva
amr			ivf			qcp
anm			jv			r3d
apc			latm			rawvideo
ape			lmlm4			rl2
applehttp		loas			rm
asf			lxf			roq
ass			m4v			rpl
au			matroska		rso
avi			microdvd		rtp
avs			mjpeg			rtsp
bethsoftvid		mlp			sap
bfi			mm			sdp
bink			mmf			segafilm
bintext			mov			shorten
bit			mp3			siff
c93			mpc			smacker
caf			mpc8			sol
cavsvideo		mpegps			sox
cdg			mpegts			spdif
daud			mpegtsraw		srt
dfa			mpegvideo		str
dirac			msnwc_tcp		swf
dnxhd			mtv			thp
dsicin			mvi			tiertexseq
dts			mxf			tmv
dv			mxg			truehd
dxa			nc			tta
ea			nsv			tty
ea_cdata		nut			txd
eac3			nuv			vc1
ffm			ogg			vc1t
ffmetadata		oma			vmd
filmstrip		pcm_alaw		voc
flac			pcm_f32be		vqf
flic			pcm_f32le		w64
flv			pcm_f64be		wav
fourxm			pcm_f64le		wc3
g722			pcm_mulaw		wsaud
g723_1			pcm_s16be		wsvqa
g729			pcm_s16le		wtv
gsm			pcm_s24be		wv
gxf			pcm_s24le		xa
h261			pcm_s32be		xbin
h263			pcm_s32le		xmv
h264			pcm_s8			xwma
idcin			pcm_u16be		yop
idf			pcm_u16le		yuv4mpegpipe
iff			pcm_u24be

Enabled muxers:
a64			ipod			pcm_s16le
ac3			ivf			pcm_s24be
adts			latm			pcm_s24le
aiff			m4v			pcm_s32be
amr			matroska		pcm_s32le
asf			matroska_audio		pcm_s8
asf_stream		md5			pcm_u16be
ass			microdvd		pcm_u16le
au			mjpeg			pcm_u24be
avi			mlp			pcm_u24le
avm2			mmf			pcm_u32be
bit			mov			pcm_u32le
caf			mp2			pcm_u8
cavsvideo		mp3			psp
crc			mp4			rawvideo
daud			mpeg1system		rm
dirac			mpeg1vcd		roq
dnxhd			mpeg1video		rso
dts			mpeg2dvd		rtp
dv			mpeg2svcd		rtsp
eac3			mpeg2video		sap
ffm			mpeg2vob		segment
ffmetadata		mpegts			sox
filmstrip		mpjpeg			spdif
flac			mxf			srt
flv			mxf_d10			swf
framecrc		null			tg2
framemd5		nut			tgp
g722			ogg			timecode_v2
g723_1			pcm_alaw		truehd
gif			pcm_f32be		vc1t
gxf			pcm_f32le		voc
h261			pcm_f64be		wav
h263			pcm_f64le		webm
h264			pcm_mulaw		wtv
image2			pcm_s16be		yuv4mpegpipe
image2pipe

Enabled protocols:
applehttp		md5			rtmps
cache			mmsh			rtmpt
concat			mmst			rtmpte
crypto			pipe			rtp
file			rtmp			tcp
gopher			rtmpe			udp
http

Enabled filters:
abuffer			deshake			nullsrc
abuffersink		drawbox			overlay
aconvert		fade			pad
aevalsrc		fieldorder		pixdesctest
aformat			fifo			rgbtestsrc
amovie			format			scale
anull			gradfun			select
anullsink		hflip			setdar
anullsrc		hqdn3d			setpts
aresample		lut			setsar
ashowinfo		lutrgb			settb
blackframe		lutyuv			showinfo
boxblur			movie			slicify
buffer			mp			split
buffersink		mptestsrc		testsrc
color			negate			transpose
copy			noformat		unsharp
crop			null			vflip
cropdetect		nullsink		yadif
delogo

Enabled bsfs:
aac_adtstoasc		mjpeg2jpeg		mp3_header_decompress
chomp			mjpega_dump_header	noise
dump_extradata		mov2textsub		remove_extradata
h264_mp4toannexb	mp3_header_compress	text2movsub
imx_dump_header

Enabled indevs:

Enabled outdevs:

License: GPL version 2 or later
Creating config.mak and config.h...
config.h is unchanged
libavutil/avconfig.h is unchanged
</pre>