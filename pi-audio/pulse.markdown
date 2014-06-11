# Defunct notes on Pulseaudio -- ignore

Tis is the SECOND dump of notes about trying to get Pulse working. The first is [here](pulseaudio.html).

ALSA works now, but it appears to be exclusive -- can't play more than one sound on a given device at once. This will lead to contention, so next I need a mixing layer on top of ALSA. That's PulseAudio.

`sudo apt-get install pulseaudio libpulse-dev pulseaudio-module-bluetooth-dbg pulseaudio-module-raop-dbg pulseaudio-module-zeroconf-dbg pulseaudio-utils-dbg`

Also, make sure your user is added to groups `audio` and `pulse-access`. 

aaaaand the first thing thsi did is make ALSA ineffective. Even if PA wasn't running before. ALSA should not be sending things to pulse (yet!) How to make that stop?

First way to make that stop is to add

```
autospawn = no
```

to `etc/pulse/client.conf`.

(but why is ALSA trying to send to pulseaudio anyway? Pulse isn't mentioned on the devices I'm talking to, and removing the pulse device doesn't affect it.)



I added this line to `/etc/pulse/daemon.conf`:

```
default-sample-rate = 48000
```

as that is the native rate of the USB devices (and as we configured ALSA earlier.)

Now why is pulseaudio both not seeing anything and hanging everything up? Is there a list of available modules?

`pulseaudio -nc` gave error messages about dbus:

```
W: [pulseaudio] server-lookup.c: Unable to contact D-Bus: org.freedesktop.DBus.Error.NotSupported: Unable to autolaunch a dbus-daemon without a $DISPLAY for X11
W: [pulseaudio] main.c: Unable to contact D-Bus: org.freedesktop.DBus.Error.NotSupported: Unable to autolaunch a dbus-daemon without a $DISPLAY for X11
```

I thought, "i'm going to run headless anyway," so I went to `/etc/pulse/default.pa` and commented the bit about `module-dbus-protocol`. But this did not stop the warnings about dbus that I got when launching `pulseaudio -nC`...

In `pulseaudio -nC` I ran `load-module module-udev-detect` and it detected some modules ... and promptly started dropping USB data (keystroke) left and right... wow.

Then [this page says][dbus] said dbus is necessary to get the hardware list! Gee, I hadn't even tried playing sound after starting an X session. dbus doesn't start without X? butbutbut dbus IS RUNNING according to `ps` and init.d status! *So why is pulse not connecting to the running instance of dbus?*

[dbus]: http://rudd-o.com/linux-and-free-software/how-to-make-pulseaudio-run-once-at-boot-for-all-your-users

To confirm, I undid my changes to default.pa, and, started X. On loading X, I found that a PulseAudio deaemon started and immediately ate all the CPU. However it wasn't dropping keystrokes, yet. Running `pacmd` told me "daemon not responding." the first time, but worked a second time; yet it had only found the builtin sound card.

`sudo pulseaudio --system -nC -F /etc/pulse/default.pa`

This spits out a bunch of errors about failing tp load a working profile, in module-alsa-card, for both my USB devices, forever, looping and at 100% CPU utilization. Ah!

So sounds like the answer is let's make a PA config from scratch. `pacmd` doesn't even run in system mode so I guess I'll work in user mode. With verbose output: `pulseaudio -vvv -nC`

Interesting errors here:

```
>>> load-module module-alsa-sink device=living_room name=living_room sink_name=living_room
D: [pulseaudio] alsa-util.c: Trying living_room with SND_PCM_NO_AUTO_FORMAT ...
D: [pulseaudio] alsa-util.c: Managed to open living_room
I: [pulseaudio] alsa-util.c: cannot disable ALSA period wakeups
D: [pulseaudio] alsa-util.c: Maximum hw buffer size is 5461 ms
I: [pulseaudio] (alsa-lib)pcm_hw.c: SNDRV_PCM_IOCTL_HW_PARAMS failed (-5)
I: [pulseaudio] (alsa-lib)pcm_hw.c: SNDRV_PCM_IOCTL_HW_PARAMS failed (-5)
I: [pulseaudio] (alsa-lib)pcm_hw.c: SNDRV_PCM_IOCTL_HW_PARAMS failed (-5)
I: [pulseaudio] (alsa-lib)pcm_hw.c: SNDRV_PCM_IOCTL_HW_PARAMS failed (-5)
D: [pulseaudio] alsa-util.c: Set neither period nor buffer size.
I: [pulseaudio] (alsa-lib)pcm_hw.c: SNDRV_PCM_IOCTL_HW_PARAMS failed (-5)
I: [pulseaudio] alsa-util.c: snd_pcm_hw_params failed: Input/output error
```

Now completely frustrated over ALSA no longer playing with pulseaudio installed. Or is it after the firmware update???? Argh.

Uninstalling pulseaudio...now. 
