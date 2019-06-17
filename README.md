
<b>UBoot</b>

Works as is, although uboot does not support composite output and directly outputs over HDMI instead.
This explains the issue i had with the screen being white & blue for a few seconds...
I seriously thought my screen was broken.

<b>Kernel</b>

Mainline kernel is pretty good and supports most of the hardware features except
for one critical flaw : it currently does not support composite output.
The Retrostone's LCD is plugged over composite and thus, we are stuck with Legacy kernel 3.4.

The legacy kernel has a few (a lot) of issues :
- The sound driver is completely awful, constantly underruns and sometimes outputs no sound.
It doesn't seem to support resampling either so we are stuck with one frequency.
I chose 48000hz stereo.
- While the DRM mali module works, it requires a proprietary firmware blobs, which has a dependency
on glibc. This makes it problematic if we insist on using uclibc or musl (see buildroot on why
i can't use glibc).


<b>Display</b>

<img src="retrostone_smsplusgx.jpg">
<i>SMS Plus GX running over fbdev (Currently not using SDL for now)</i>
<br></br>
RetrOrangePi sets the resolution to 672x448.
However, this is not actually the maxed out resolution you can get without black borders.
The maximum resolution i could get is 680x448. Offsets are x: 40 and y : 18.
<br></br>
There are two displays opened, first one is the one we draw to (where the resolution is actually set)
and the second one, which seemingly refuses to change its resolution over composite, is for the final output.
(Linux says resolution of that one is 800x480)
<br></br>
RetrOrangePi sets the bitdepth to 32, which is too high for this device and also won't allow OpenDingux ports
as easily.
My distribution will set it to RGB565, which is value 10, not 4 as reported by the Sunxi wiki...
<br></br>
Emulators don't manually set the resolution so it's somewhat flexible but i still
need to figure out how to use the HDMI out.
<br></br>
<b>3D Acceleration</b>
<br></br>
Mali userspace blobs are built against glibc as i said above.
It's possible to use the mainline kernel and rebuild mesa with the soon-to-be-merged lima driver but
mainline kernel does not support CVBS so we're out of luck.
Sunxi devs, if you are looking at this, hopefully you can find someone gentle enough to get this working over CVBS...

For now, we will do without 3D acceleration (and just using the DRM interface), this is what most opendingux devices did after all.
(Besides, the Mali 400 isn't exactly the most powerful GPU out there)

<b>Sound</b>

The sound driver is garbage : It constantly underruns, sometimes outright refusing to output any sound and doesn't support resampling of differing frequencies.
I heard that mainline has a more robust implementation but again, no luck for us here...
I chose 48000hz stereo as the default and emulators target that.

 <b>Input</b>
 
RetrOrangePi handles input by creating a virtual device with uinput, and it does this through a python script.
I did something similar but in C : https://github.com/retrostone-dev/remap-input
The program maps the GPIO keys as a virtual keyboard, especially to allow OpenDingux ports.

However, i ended up directly patching SDL in order to avoid more input latency by directly reading from the GPIO pins.
The difference is quite noticeable btw.

<br></br>
<b>Userspace</b>
<br></br>
Because we are using an odd resolution, we need to export an environment variable to tell SDL to accept it as it is.
Otherwise it will refuse to display anything.
