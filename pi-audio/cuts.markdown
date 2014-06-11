
Specifically, I created a file `/etc/udev/39-usb-alsa.rules`, with the contents:

```
KERNEL=="controlC[0-9]*", DRIVERS=="usb", PROGRAM="/usr/bin/alsa_name %k %b", NAME="snd/%c{1}" KERNEL=="hwC[D0-9]*", DRIVERS=="usb", PROGRAM="/usr/bin/alsa_name %k %b", NAME="snd/%c{1}" KERNEL=="midiC[D0-9]*", DRIVERS=="usb", PROGRAM="/usr/bin/alsa_name %k %b", NAME="snd/%c{1}" KERNEL=="pcmC[D0-9cp]*", DRIVERS=="usb", PROGRAM="/usr/bin/alsa_name %k %b", NAME="snd/%c{1}"
```

and a Perl script at `usr/bin/alsa-name` with the numbers:









Hopefully that will cause my USB devices to be stable.

Now what........ I can 

