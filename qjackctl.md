# Configuring qjackctl jackaudio environment

## System configuration

I like to occassionally use different outputs of my system. So I'd like them to be available to the patchbay. 

I use manjaro xfce variant, so I added qjackctl to the "Session and Startup" settings in the Graphical Settings Manager for Xfce.

After getting a basic setup of qjackctl to start and run. I use Carla to organize my patchbay connections.

Then in the setup > options tab of qjackctl I add the following: 

/home/user/jack-pre-start.sh
```
#!/bin/bash
pacmd suspend true
```
/home/user/jack-post-start.sh
```
#!/bin/bash

pactl load-module module-jack-sink client_name=jack_out
pactl load-module module-jack-source client_name=jack_in
pactl set-default-sink jack_out
pactl set-default-source jack_in
nohup alsa_out -j spdiff -d hw:Generic,1 -c 2 &
nohup alsa_out -j Motherboard -d hw:Generic,0 -c 6 &
nohup alsa_in -j UR22Microphone -d hw:UR22mkII &
nohup aj-snapshot -d 20211122_aj-snapshot &
```
/home/user/jack-pre-stop.sh
```
#!/bin/bash
pactl unload-module module-jack-source
pactl unload-module module-jack-sink
```
/home/user/jack-post-stop.sh
```
#!/bin/bash
pacmd suspend false
```

## Aj-snapshot
aj-snapshot can create a text file describing your patchbay connections. Any time you update your patchbay, you can take a new snapshot. Every second and configurable, aj-snapshot will replace missing connections.
alsa_out and alsa_in create jack connections for other audio devices on your system, because qjackctl will only start jack with one audio device. An alternative is zita-a2j and zita-j2a which did not work well for me.

Good luck.
