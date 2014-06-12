Chris's Dynamic Compressor
===============================
Plugin for Audacity

*Update:* 1.2.7 beta 1 version released. Interesting new feature! See below.

I've written a program that makes it easier to listen to classical music, or other music that has a wide range of volumes, at low volumes or in high noise conditions (such as in your car) so that you can still hear the soft parts. I've also written a plugin version, designed to be used with the free audio editor [Audacity](http://audacity.sourceforge.net/). You can read about the background and rationale of the plugin [here](rationale/index.html).

For you to hear it in action, I've provided three 30 second clips of the fourth piece from Debussy's Children's Corner suite, The Snow is Falling. They use version 1.1 of the compressor, so results will differ slightly, and the settings are named differently. The [first](../../projects/compressor/snow-unc.mp3) has no adjustment, the [second](../../projects/compressor/snow-c1.5.mp3) has compression applied with both rise and fall speed at the default 1.5 dB/s, and the [third](../../projects/compressor/snow-c5.mp3) has compression applied with fall and rise speed set to a more aggressive 5 dB/s.

It was a lot of fun to program. I hope a lot of people get some good use out of it. I hope it allows me, and possibly you, to enjoy our classical music more often, and to more easily share it with our friends.

### Download and Use

My program currently comes in two versions. One is a standalone program, both with a GUI and console interface. It's useful for batch processing, and allows you to directly compress to/from MP3s (Lame required). (Audacity doesn't currently support batch processing using a Nyquist plugin.) Download version 1.2.6 [here](../../projects/compressor/Compressor-dotnet.zip). It requires the .NET framework version 3.5 (or Mono 1.9). If you're using the GUI version, check out the commandline version for explanations of the parameters. People have reported problems using this version on Vista and Windows 7. (Specifically, the output is playing back at double speed.) I doubt the OS is the problem, but I've had no reports of problems on XP. You can try it anyway, or you can contact me and help me debug it and fix the problem.

The other is an Audacity plugin, currently at version 1.2.6 (or, below, a beta version 1.2.7). To use it, [download](http://audacity.sourceforge.net/download/?lang=en) and install Audacity. Download the [plugin source](../../projects/compressor/compress.ny) (i.e. right click on that link and select "Save target as..." or your browser's equivalent), and put it in your audacity plugins directory, which should be under the main install directory. On windows, that's usually C:\Program Files\Audacity\plugins. On unix, /usr/[local/]share/audacity/plugins. On OS X, /Applications/Audacity/plug-ins. Your browser, if it sucks, might insist on saving the file with a ".txt" extension behind your back. If it does, you'll have to remove this to get the plugin installed.

[1.2.7 beta 1](../../projects/compressor/compress-b1.ny) is a new version of the audacity plugin with an experimental feature enabled by default (at the bottom of the parameters screen). It might give better results, especially for very low volume listening (which includes listening in noisy environments, such as in cars). It adjusts for the extra sensitivity people have to frequencies in the 3-7 kHz range, i.e. bright or brassy sounds. It doesn't change the tone, but it makes bright sounds softer than it otherwise would, to give a perceptually more even volume, even though the actual amplitude is less equal. Try it out if you like. Let me know what you think. (I tried a feature to compensate for the lesser sensitivity to bass sounds, but the results were terrible for some reason.)

If your music is on CD, use a CD ripping program to make a .wav file of each track. You can encode the wav file (to mp3, ogg, etc.) after you've applied the dynamic compression. Use Audacity to open the sound file. Select the whole thing, then go to the Effects menu, to the bottom at Plugins (or below the horizontal divider), and select "Compress dynamics...". Adjust the settings to your liking, and click OK. To save your changes, you need to go to File > Export wav or Export mp3. File > Save will not do what you want it to.

The plugin version has advanced options (including separate attack/release speeds) that can be activated by downloading [this version](../../projects/compressor/compress-adv.ny), or by following the instructions in the .ny file. If you're used to having finer control over your compressor, you might consider these. (They're explained in [the tutorial](options-explained/index.html).)

For the most part, the default settings will work just fine. The first setting is the main one, letting you choose how heavy or light to compress.
### Tutorial

[Here.](options-explained/index.html)

### Changes in 1.2.6

For the plugin: Hide a bunch of advanced options by default and replace them with a single option that should be sufficient for 99% of uses. Clarify documentation a bit.

For the standalone: Throw an error when the WAV file isn't in the expected format. Also, try to handle long RIFF subchunks before the data chunk better.

### Changes in 1.2.5

1.2.2 introduced a problem where memory usage was very high, so it probably didn't work on very long inputs. It didn't work on mono inputs either.

Let's just pretend 1.2.4 never happened.

### Changes in 1.2.3

1.2.2 was broken on longish inputs (more than a few minutes) due to a bug in Nyquist. I was able to work around the bug.

### Changes in 1.2.2

Two bugs were fixed. One caused tracks to be truncated sometimes after a period of silence in the track. The other caused inappropriate volume gains on certain parts of the track, usually at the end. (Specifically, the compression was being applied about 1/30 sec too soon.) Neither of these bugs were present in the program version.

### Changes in 1.2.1

A simple noise gate is added. The floor is specified in decibels now, instead of linear.

### Changes in 1.2

A completely different compression algorithm is used, based on parabolas instead of lines. It strikes a better balance between fast and slow changes, by varying its rate of change so that the envelope doesn't leave so much empty space (i.e. low-volume sections) without losing too much transparency. You get a couple more parameters to play with. You can emulate the behavior of the old version by setting the exponent parameters to 1. The audacity version has much better memory usage (and thus, speed) on larger inputs than the previous version.

### Changes in 1.1

- Bugfix. It caused some areas to be amplified way too much on files, especially when the rise and fall speed were too high.
- Renamed existing parameters to more conventional things for compressors.
- Added a noise floor parameter, which lets you leave alone (apply a constant gain) parts of the audio below a threshold.
- Added a compression ratio parameter, which lets you not bring everything all the way up to the same level, and also do weird things with ratios outside the normal range.

### How it works (technical details)

First you resample down by a factor of 1500 or so using the max and abs functions to get a representation of the volume of the sound at any given point (the input envelope); Then you construct a compression envelope around the input envelope that "hugs" it in just the right way, and then you drag up the compression envelope points to 0 dB (and the volume of the input sound with it), i.e. you multiply the input sound by (1 - 1 / the compression envelope) * the compression ratio (or something like that—you get the idea, I think).

My compressor calculates its compression envelope thus: It first calculates a "paraboloid", which is what I call the function

```
f(x) = x > 0 ? a1*x^b1 : a2*x^b2
```

where `a1,2` and `b1,2` are parameters to the compressor. `b1` and `b2` are set by default to 4 and 2 respectively, which I think is the key to the compressor's effectiveness. Then it tries to fit sections of this paraboloid to hug, as closely as possible, the input envelope without going under it (which would cause clipping). It turns out there's a unique fitting that's the best, and it's pretty simple to calculate using lookahead with a trial-and-error technique.

And that's basically it. In versions earlier than 1.2, I had the simpler idea of using straight lines instead of curves, which gives OK results, but not quite as nice.