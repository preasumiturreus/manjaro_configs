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
