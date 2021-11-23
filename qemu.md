# Qemu Configs

My most common guest is windows 10 run with a manjaro host started with virt-manager. 

Switching my qemu/libvirt configuration to jack connections required removing the magic pulseaudio xml:

### The old
```
    <qemu:arg value="-audiodev"/>
    <qemu:arg value="pa,id=pa1,server=/run/user/1000/pulse/native,in.latency=10000,out.latency=10000"/>
    <qemu:arg value="-device"/>
    <qemu:arg value="ich9-intel-hda,bus=pcie.0,addr=0x1b"/>
    <qemu:arg value="-device"/>
    <qemu:arg value="hda-micro,audiodev=hda"/>
    <qemu:arg value="-audiodev"/>
    <qemu:arg value="pa,id=hda,server=unix:/run/user/1000/pulse/native"/>
    <qemu:arg value="-audiodev"/>
```
### The new
    <qemu:arg value="-audiodev"/>
    <qemu:arg value="jack,id=win10large,in.format=s32,out.format=s32,in.frequency=48000,out.frequency=48000,in.fixed-settings=true,out.fixed-settings=true,timer-period=512"/>
    <qemu:arg value="-device"/>
    <qemu:arg value="ich9-intel-hda"/>
    <qemu:arg value="-device"/>
    <qemu:arg value="hda-duplex,audiodev=win10large"/>
    <qemu:env name="PIPEWIRE_RUNTIME_DIR" value="/run/user/1000"/>

### Problem!
Sound wasn't working from the win10 guest. The Control Panel > Sound program showed output. 
Checking `/var/log/libvirt/qemu/virtualmachinename.log` showed 
```
jack: E: Cannot create thread res = 1
jack: E: JackMessageBuffer::Create cannot start thread
jack: E: Cannot create message buffer
jack: E: Cannot create thread res = 1
jack: E: Cannot start Jack client listener
jack: E: Cannot start channel
jack: E: JackShmReadWritePtr1::~JackShmReadWritePtr1 - Init not done for -1, skipping unlock
jack: E: JackShmReadWritePtr::~JackShmReadWritePtr - Init not done for -1, skipping unlock
jack: E: JackShmReadWritePtr::~JackShmReadWritePtr - Init not done for -1, skipping unlock
```

### Magic sauce
[disabling seccomp_sandbox on libvirt/qemu.conf](https://www.reddit.com/r/VFIO/comments/ga4cp3/qemu_native_jack_audio_support_vfio/fsxl7og/) and `sudo systemctl restart libvirtd` did the trick. Now, patchbay clients appear in Carla/patchage or whatever patch client you use.
The next issue was shown in `/var/log/libvirt/qemu/virtualmachinename.log`:
```
jack: E: Cannot use real-time scheduling (RR/5) (1: Operation not permitted)
jack: E: JackClient::AcquireSelfRealTime error
jack: JACK output configured for 48000Hz (512 samples)
```

The solution was following [this level1techs post](https://forum.level1techs.com/t/qemu-native-jack-audio-support/156494/38)
To summarize, to create override .confs for a few systemd services. Which was to follow the [arch wiki](https://wiki.archlinux.org/title/Systemd#Editing_provided_units) about drop-in files.
Create the following and restart libvirtd.service: 
```
~/.config/systemd/user/pulseaudio.service.d/override.conf
~/.config/systemd/user/dbus.service.d/override.conf
/etc/systemd/system/libvirtd.service.d/override.conf
```
Each containing
```
[Service]
LimitRTPRIO=95
LimitNICE=-16
LimitMEMLOCK=infinity```

Since I start jack server using qjackctl with dbus option unchecked, jackd is run as a separate dbus-unaware process, I think. Making an override.conf for dbus.service and pulseaudio.service may be optional.

I have previously tried running jack with Cadence and had issues with poor performance and xruns. I suspect a dbus misconfiguration, and while I don't understand the RTPRIO option, maybe jackdbus may perform better now.
