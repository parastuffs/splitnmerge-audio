# Split'n'Merge Audio
The aim of this project is to find a way to merge different sources of audio into a virtual mic.

## Jack
First step: try to install and run a JACK server.

https://wiki.archlinux.fr/Jack


If you find the message
```Could not open component .so '/usr/lib/jack/jack_firewire.so': libffado.so.2: cannot open shared object file: No such file or directory```
when starting JACK daemon, you might need to install a firewire driver (even if you don't have a firewire input): `libffado`.

A tad complex to put in place, let's look into Pulseaudio

## Pulseaudio
The idea is to create a virtual mic `virtualMic` that merges the real mic and the music, as well as a virtual output `virtualSp` so that we can ear the music too.

1. Virtual sinks:
```
pactl load-module module-null-sink sink_name=virtualMic sink_properties=device.description=virtualMic
pactl load-module module-null-sink sink_name=virtualSp sink_properties=device.description=virtualSp
```

2. Add a loopback to `virtualMic`
```
pactl load-module module-loopback sink=virtualMic
```

3. Redirect the monitor of `virtualSp` to `virtualMic`
```
pactl load-module module-loopback sink=Virtual1 source=Virtual2.monitor
```

4. Redirect the monitor of `virtualSp` to the real speakers as well.
```
pactl load-module module-loopback sink=alsa_output.pci-0000_00_1b.0.analog-stereo source=Virtual2.monitor
```
Where the name of the sink to which I want to redirect is listed through
```
pactl list short sinks
0       alsa_output.pci-0000_00_1b.0.analog-stereo      module-alsa-card.c      s32le 2ch 44100Hz       RUNNING
1       Virtual1        module-null-sink.c      float32le 2ch 44100Hz   RUNNING
2       Virtual2        module-null-sink.c      float32le 2ch 44100Hz   IDLE
```
We can also give the last `pactl` command the id of the sink instead of its name.

5. In `pavucontrol`, set and check the following:
- Input Devices: default fallback on `Monitor of virtualMic`
- Output Devices: default fallback to your speakers.
- Recording:
  - Loopback to `virtualMic` from real mic
  - Loopback to `virtualMic` from Monitor of `virtualSp`
  - Loopback to speakers from Monitor of `virtualSp`
  - Recodring app (Discord, Audacity, etc.) from Monitor of `virtualMic`
**Check that they are all unmuted!**
- Playback:
  - Loopback of real mic on `virtualMic`
  - Loopback from Monitor of `virtualSp` on `virtualMic` --> music to your virtual mic, this is the level to adjust for the broadcast only
  - Loopback from Monitor of `virtualSp` on speakers --> you can still ear your own music, changing this level will only update the volume for *you*
  - Output of your discussion app (Discord, etc.)  on speakers
  - Music on `virtualSp` --> changing this level will modify the volume for you *and* the broadcast

Check the references for more details.


## References
[Adding Ambient Sounds to your Discord Server On LInux](https://phpboyscout.uk/adding-ambient-sounds-to-your-discord-server-on-linux/)
[Share an audio playback stream through a live audio (video) conversation like Skype](https://askubuntu.com/questions/421014/share-an-audio-playback-stream-through-a-live-audio-video-conversation-like-sk)
