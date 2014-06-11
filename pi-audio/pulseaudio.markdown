# Really, ignore this and go back to the [main page](index.html).

#### PulseAudio

One thing I don't understand is the proliferation of Linux audio solutions which might or might not all depend on eaceh other. ALSA, libao, JACK, PulseAudio, and so on? Apparently PulseAudio is the new thing. I got pretty confused by the situation last time I poked around it.

However, some of my desiderata are for mixing and lack of contention: if one device is broadcasting silence, and another wants to bradcast something else, there should be no interruption. So some audio server seems necessary.

Here's a page on getting Raspberry Pi and PulseAudio playing together: [Raspberry Pulse][rpulse].

[rpulse]: http://www.foell.org/justin/raspberry-pulse/

I tried to follow these instructions but discovered that PA had disabled the combination of system mode and module loading:

```
pi@raspberrypi ~ $ tail /var/log/messages
Mar 27 04:34:49 raspberrypi kernel: [   30.896640] Adding 102396k swap on /var/swap.  Priority:-1 extents:1 across:102396k SSFS
Mar 27 04:34:49 raspberrypi /usr/sbin/gpm[2146]: *** info [daemon/startup.c(131)]: 
Mar 27 04:34:49 raspberrypi /usr/sbin/gpm[2146]: Started gpm successfully. Entered daemon mode.
Mar 27 04:34:54 raspberrypi pulseaudio[2471]: [pulseaudio] main.c: Running in system mode, but --disallow-module-loading not set!
Mar 27 04:34:54 raspberrypi pulseaudio[2471]: [pulseaudio] main.c: Running in system mode, forcibly disabling SHM mode!
Mar 27 04:34:54 raspberrypi pulseaudio[2471]: [pulseaudio] main.c: Running in system mode, forcibly disabling exit idle time!
Mar 27 04:34:54 raspberrypi pulseaudio[2473]: [pulseaudio] main.c: OK, so you are running PA in system mode. Please note that you most likely shouldn't be doing that.
Mar 27 04:34:54 raspberrypi pulseaudio[2473]: [pulseaudio] main.c: If you do it nonetheless then it's your own fault if things don't work as expected.
Mar 27 04:34:54 raspberrypi pulseaudio[2473]: [pulseaudio] main.c: Please read http://pulseaudio.org/wiki/WhatIsWrongWithSystemMode for an explanation why system mode is usually a bad idea.
Mar 27 04:36:19 raspberrypi pulseaudio[2473]: [pulseaudio] protocol-native.c: Denied access to client with invalid authorization data.
```

Okay, so.... perhaps not use system mode, but automatically start the daemon under my user?

In any case, still getting authorization denied messages:

```
Mar 27 04:44:17 raspberrypi pulseaudio[2473]: [pulseaudio] protocol-native.c: Denied access to client with invalid authorization data.
```

This was resolved by:

```
pi@raspberrypi ~ $ sudo gpasswd -a pi pulse-access
Adding user pi to group pulse-access
```

Additionally /var/log/messages contains lovely messages like

```
Mar 27 04:46:48 raspberrypi rsyslogd-2177: imuxsock lost 224 messages from pid 2473 due to rate-limiting
Mar 27 04:46:50 raspberrypi rsyslogd-2177: imuxsock begins to drop messages from pid 2473 due to rate-limiting
```

So something is stuck?

Further, when starting the CLI using `pulseaudio -nC` as described in "first steps" I get 0 sound cards detected and no sinks... same when trying as root (`sudo pulseaudio --system --nC`)

starting pulse autio with verbose logging (`pulseaudio -D --log-level=debug`) spids a mass of information out to `/var/log/messages/`.

I restarted, then tried just running `alsamixer`. It showed that ALSA was outputting to pulseaudio at level 63 (interestingly, setting any other level just got reset to 63.

Is this chasing its own tail? PulseAudio using ALSA as output while ALSA uses pulseaudio as output? PulseAudio server started when I started alsamixer.

Okay, so... have a look at [this](1000px-Pulseaudio-diagram.svg.png). The expectation is that apps talk to ALSA which sends its input to PulseAudio which mixes and sends audio back to a different aspect of ALSA which talks to the sound card??? This is supposed to be the normal setup??? And it appears that in the default install of PulseAudio, ALSA thinks it is to send things to PulseAudio while PulseAudio thinks it is to send things to ALSA without specifying that ALSA not send things back. Or something.




###