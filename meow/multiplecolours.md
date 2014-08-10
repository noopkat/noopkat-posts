#3D printing in multiple colours with one extruder

Dual extruder printers are pretty sweet. They have two different independent extruder assemblies attached to the same linear actuators. This means that it's fairly easy to have gcode that controls the two print heads separately. 

Having two extruders working in the same print job is really handy for a number of reasons. Loading a different colour filament in each is a great way to create prints that are multicoloured. However, there are others ways to print in multiple colours, if a little more limited than a dual setup.

Here is a print I did recently, the npm logo:

![npm logo]()

And a bracelet:

![stretchlet bracelet]()

So how was this achieved? If your guess is pausing a print mid job, unloading the current filament and switching it for another colour filament, you're on the money!

There are a few ways to do this:

1. If you're controlling your print with software such as Repetier Host, Pronterface, Octoprint, or Cura, there is a pause/resume button available in the UI while a print job is running. Just watch your print until you're ready to change colours, pause, swap out, and resume.

2. If watching a print like a hawk is not your thing, or you're padantic about changing colours at the right point, you can automatically inject a few added lines into your gcode while slicing your stl. Some slicing software, such as Cura, have plugins specifically to do this. Just set the height you want to change filament at, and where the extruder head should move to while paused. Easy peasy! I tried the ['Change Filament at Z' plugin for Cura](https://github.com/smorloc/CuraPlugins/blob/master/ChangeFilamentAtZ.py), but was a little unhappy with some incompatibilities with my own printer. I have forked the code and fixed a couple of things to suit my needs, so [feel free to check it out](https://github.com/noopkat/CuraPlugins/blob/custom-tweaks/ChangeFilamentAtZ.py) and install! Continuing the print is as simple as clicking 'resume', similar to solution 1.

3. Similar to solution 2, but manually write the gcode yourself. It's pretty simple if you have an example to work with. This will depend on your chosen slicer commenting each new layer within the gcode. This makes finding the correct layer buried in the gcode a *lot* easier! Here is an example of a 'change filament' block you can paste and tweak, just before the chosen layer begins:

```
M83 ; turn on relative movement for extruder
G1 E-5.000000 F6000 ; retract filament 5mm
G1 X0.000000 Y0.000000 Z4.000000 F9000 ; home X and Y axis leave Z at current height
M84 E ; release extruder stepper motor from 'holding' position
@pause ; pause print!
G1 X0.000000 Y0.000000 F9000 ; upon resume, rehome X/Y in case position was bumped out
G1 X69.080000 Y58.560000 F9000 ; move back to next layer starting position
G1 E0 F6000 ; reset extruder, ready to push out plastic again
G1 F9000 
M82 ; set extruder movement back to absolute ready for next layer
```

So how do you even change the filament? The plugin/script auto retracts a little to get you started which is nice. But there's no magic - just pull it out and manually load in the next colour as you normally would before kicking off a print from the beginning. There *are* two different effects you can try though when doing this.

The first is a **clean colour change**. This means that after your print is paused and new colour filament loaded, the filament is fed continuously (via your 3D printer software) until the new colour feeds out cleanly and without the previous colour mixed into it. This method was used for the npm logo, and the stretchlet bracelet (photos above).

The second is what I call a **gradient or gradual colour change**. The difference is that instead of feeding the old colour through until the new colour appears in the previous method, you simply resume the print immediately after swapping over the filament. No manual feeding. Theß effect you'll see is more of a blurred or smooth change in colour over the next 5 - 10 layers, depending on the resolution you're printing at.

You can see this effect on the following [fan grate](http://www.thingiverse.com/thing:393764) I printed for my Printrbot:

![fan grate]()

Looks pretty cool, huh?

Give it a try and share the results if you like!