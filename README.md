# manjaro_configs
notes on configuring manjaro

Stock manjaro provides pulseaudio server for sound output. It seemed somewhat limiting so I decided to try jack.

To perform simultaneous output to multiple hardware devices, like a USB Dac and the motherboard sound card, in the jack configuration (like cadence or qjackctl) choose one as the output device. Next, use the alsa_out utility to create a new jack device that can be connected to the application's outputs.

For instance, 

```
╔═╗  0 16:10:39 [user@user-manjaro /proc/asound]                        
╚═╩═ $ cat /proc/asound/cards
 0 [HDMI           ]: HDA-Intel - HDA ATI HDMI
                      HDA ATI HDMI at 0xfc860000 irq 101
 1 [Generic        ]: HDA-Intel - HD-Audio Generic
                      HD-Audio Generic at 0xfcb00000 irq 103
 2 [UR22mkII       ]: USB-Audio - Steinberg UR22mkII
                      Yamaha Corporation Steinberg UR22mkII at usb-0000:06:00.3-2.1, high speed
 3 [Device         ]: USB-Audio - USB Modi Device
                      Schiit Audio USB Modi Device at usb-0000:0b:00.3-1, high speed
```

![Screenshot_2021-11-17_16-21-12](https://user-images.githubusercontent.com/89953202/142284813-f4b2b7c7-4931-451e-a56e-f175ef097596.png)

The USB Modi Device is selected to be the jack server's 'system playback_1 and playback_2'. Now, use alsa_out to create a second output.

```
alsa_out -j motherboard_out -d hw:Generic
```

Another option to alsa_out is zita_a2j and zita_j2a. [Here](https://kokkinizita.linuxaudio.org/linuxaudio/zita-ajbridge-doc/quickguide.html)

I was having a problem with my optical output (S/PDIFF) after installing jackaudio. To fix, run alsamixer, and select the motherboard's sound card. Using arrow key, select the S/PDIFF playback channel, and press 'M' to unmute. A good explanation of alsamixer is available at [linuxaudio wiki](https://wiki.linuxaudio.org/wiki/alsa_and_kxstudio).
![alsamixer config](https://github.com/preasumiturreus/manjaro_configs/blob/main/Screenshot_2021-11-19_12-24-20.png?raw=true)

I hope to get better performance from my motherboards sound chip than the usb DAC. The downside to this is my 5 channel speaker set now is on the same sound card as my headphones connected to the amplifier to the SPDIFF port.

The S/PDIFF output on the motherboard sound card vs the 5 (?) channel analog outputs can be described as hw:Generic,1 vs hw:Generic,0 respectively.

Run the command `alsa_out -j Mobo_out -d hw:Generic,0 -c 6` . Channels 5 and 6 seem to be the same, the center channel.
The following does not seem to work `zita-j2a -j Mobo5Channel -d hw:Generic,0 -c 6`, it spams "Starting synchronisation." According to the man page this indicates a major problem. Running zita-j2a with tracing '-v' spams `Alsa_pcmi: error on playback pollfd.` between each starting synchronisation.

`zita-a2j -j Mobo -d hw:Generic,0` would correspond to a 2-channel microphone input to jackaudio from the motherboard.

## pulseaudio resource
https://shallowsky.com/linux/pulseaudio-command-line.html
