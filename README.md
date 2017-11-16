# kodi-snapcast


Running snapcast client and server on kodi (openelec) together with pulseaudio pipe

## Getting Started

No kodi addons (any help to create them are highly appreciated ) 
only manual setup via ssh for now.

Whole idea of the project is to watch movies locally in one room, but music should be delivered to multiple rooms.

to do so we need to setup 3 components.

   * sound should go to pulseaudio's pipe
   * snapserver should read sound from pipe
   * snapclient should connect to server and play sound to soundcard.

### Prerequisites

Running for couple of months on current setup

Software [Forum thread to download the image](http://www.orangepi.org/orangepibbsen/forum.php?mod=viewthread&tid=648 "")

```bash
##############################################
#                  OpenELEC                  #
#             http://openelec.tv             #
##############################################

OpenELEC (community) Version: devel-20160806192003-r22886-g5828aec
OpenELEC git: 5828aec7bf38484ab49e1019c09d8b14164b9ccd
OpenELEC:~ # uname -ra
Linux OpenELEC 3.4.112 #1 SMP PREEMPT Sat Aug 6 19:15:27 CEST 2016 armv7l GNU/Linux
```
Hardware: orangepipc AllWinner H3 SoC


### Installing



#####1. Pulseaudio setup (on openelec image it is running in system mode)

```bash
ps aux | grep pulse
  265 root       0:00 /usr/bin/pulseaudio --system
```
from [here](https://github.com/OpenELEC/OpenELEC.tv/blob/master/packages/audio/pulseaudio/config/pulse-daemon.conf.d/README "") we have 
> The main daemon.conf file is processed first, so 
options set in files under pulse-daemon.conf.d override the main file.

```bash
.config/pulse-daemon.conf.d/my.conf
```

will force pulseaudio to override config provided by image. We need it to make snapfifo file permanent. 




in the end of *new* ```system.pa``` we have two additional lines


```
load-module module-pipe-sink file=/tmp/snapfifo rate=48000 sink_name=Snapcast
update-sink-proplist Snapcast device.description=Snapcast
```


#####2. snapserver setup 
is a bit tricky, as it is require to use compiled binaries by unknown person (me) if you are lucky just use binaries provided by image. The same binaries should work on raspberry pi as architecture is armv7l, but of course it have not been tested.

Copy ```/storage/.kodi/addons/snapserver``` folder to your openelec system.

Copy ```/storage/.config/system.d``` folder to your openelec system.

```snapserver.service``` should work as it is,


#####3. snapclient setup
```snapclient.service``` must be updated according you setup and snapclient and snapserver should be enabled by systemctl.

run ```LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/storage/.kodi/addons/snapserver/lib /storage/.kodi/addons/snapserver/snapclient --list```
to get the list of available soundcards on your system.

Example output 
```
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/storage/.kodi/addons/snapserver/lib /storage/.kodi/addons/snapserver/snapclient --list
0: null
Discard all samples (playback) or generate zero samples (capture)

1: default:CARD=audiocodec
audiocodec, 
Default Audio Device

2: sysdefault:CARD=audiocodec
audiocodec, 
Default Audio Device

3: front:CARD=audiocodec,DEV=0
audiocodec, 
Front speakers

4: default:CARD=sndhdmi
sndhdmi, 
Default Audio Device

5: sysdefault:CARD=sndhdmi
sndhdmi, 
Default Audio Device

6: hdmi:CARD=sndhdmi,DEV=0
sndhdmi, 
HDMI Audio Output

7: default:CARD=audiohub
audio-hub, 
Default Audio Device

8: sysdefault:CARD=audiohub
audio-hub, 
Default Audio Device

9: rear:CARD=audiohub,DEV=0
audio-hub, 
Rear speakers

10: default:CARD=NX
SB Audigy 2 NX, USB Audio
Default Audio Device
```
so in my case ```snapclient -d -h localhost --soundcard 10```

snapclient will play sound to soundcard Nr 10 (SB Audigy 2 NX)


## Actual Usage

Start to listen any audio in Kodi, after go to [Settings/System/Audio](http://kodi.wiki/view/Settings/System/Audio "") settings and switch output to PulseAudio.

The following will happens in background:

* Kodi > PulseAudio > snapfifo

* Snapserver will read snapfifo.

* local Snapclient will connect to local snapserver and will play sound via configured soundcard to local speakers. 


### TODO

* snapclient playing sound to 2 channels only, configure system to upmix 2 channels to 6 channels as I'm already have them connected.
* automatically switch to pulseaudio soudcard if user switched to music menu in Kodi (with use of Kodi Callbacks)
