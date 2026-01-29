
# 15/01/26 - 16/01/26

I own a Thinkpad X1 Carbon Gen 7 laptop and I just love opening it up and admiring the internals. It's quite possible the best laptop I've even owned, and its so organized and thoughtfully designed, It's got captive screws for the back cover and everything on the inside is nicely laid out.

Here's a picture of how it looks on the inside.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Ff7aec21a_image.octet-stream)

Now something I've noticed is that this Thinkpad has a lot of empty space, especially right under the battery when you remove it.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2F1b42c22b_image.octet-stream)

That got me thinking. What if I put embed something inside my laptop. I thought a lot about it and was kind of lost on what to embed inside. But then I noticed the red circle inside, and I realized that it was actually the red dot on the thinkpad logo on my laptop.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Fb80e2153_image.octet-stream)

I wondered if I could somehow get the red dot to glow, to test my theory I took a flashlight and shined it on the big red dot on the inside and sure enough you could see the red dot glow.
This gave me a nice idea, what I basically make a tiny PCB that has a RGB light and then I could basically make the red dot on the thinkpad glow.

And so I began my journey for this project. I first took a picture of the insides and brought it in figma and drew dimensions using the values I measured from my calipers.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Fef32494a_image.octet-stream)

And looking back this was pretty much a waste of time because I had to manually measure so many things, and in the end I couldn't even export it as a DXF so I couldn't even use it in kicad later on. But this is what my measurements looked like.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Fb757ae60_image.octet-stream)

Since I couldn't really use this I went on fusion360 and imported this image, calibrated it and drew some outlines which I could export and use as guidelines for my PCB later.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Ffbc1db0f_image.octet-stream)


After exporting this as a DXF I hopped on KiCad and started drawing up a schematic.

Now for the actual PCB I wanted a tiny board with a MCU and an RGB LED so I began searching for some suitable ICs to use. In the end I decided to go with an ATSAMD21, as it's a very tiny QFN MCU that barely needs any supporting circuitry + It does crystal less USB. 

For my RGB LED I decided to go with a right angle LED (I think that's what they are called), instead of shooting light up or down, it shoots it sideways. This was good for me because that meant I could place the LED right next to the red dot.

Now that was pretty much it for what I initially wanted to achieve, but I thought to myself, if I'm gonna embed a PCB inside my laptop why not pack it full of sensors. So that's exactly what I did.

I searched up and added quite a few small sensors on the board. For most of my sensors I went with sensors that had I2C so I could save GPIO and more importantly keep routing easier.

All in all I added a temperature sensor, a magnet sensor, an IMU and finally a vibration sensor.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2F9c7b5611_image.octet-stream)

All of them used I2C except for the vibration sensor, which I hooked up to an analog capable pin on my MCU.

I additionally also decided to add a vibration motor just so I could get some haptic feedback. Implementing this was not very straightforward. Even though a motor is just a 2 terminal device, I can't just hook it up to a MCU pin directly and control it as it requires a lot of current, which would straight up fry the MCU. 
The solution to this problem would be to basically have the MCU control a switch that turns power on/off to the motor, and that's where MOSFETs come in. A MOSFET is basically a voltage controlled switch, and it gives you the ability to switch on/off large things with small input (not really a good description). But anyways I implemented what's known as a low side switch with an NMOS FET.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2F994402b9_image.octet-stream)

When the VIB_MOT goes high the switch basically closes and the motor is connected to ground and current can flow through it and the motor can run. And similarly when the VIB_MOT goes low (by default it's in a low state because we added a pull down resistor) the switch is off and the motor is not connected to ground, ie- current can flow through it and it wont run.
Since a motor is an inductive load (meaning it stores energy in magnetic fields), when its disconnected from ground it can suddenly output a lot of voltage in the reverse polarity, and that can potentially fry our MOSFET, to solve this we add a diode in parallel, so when it's disconnected that voltage can flow through the diode and not damage our FET.


Now after finishing all this up I added a USB port and a small LDO on board, and then moved onto assigning footprints. I went of 0402s for my passives and for my motor, USB and debug headers I went with JST_SH SMD connectors. SH series is the second smallest JST connectors you can get (atleast on lcsc I think), its 1.0mm pitch headers and are basically prefect for my use case.

This is what my finished schematic looked like:

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Fa8573d40_image.octet-stream)

## Time Spent: 4 Hours


# 17/01/26 

I started working on the PCB and the first thing I did was import the DXF I made in Fusion360 to basically get a visual idea of how I should layout my components.

![](https://cdn.2008000.xyz/cdn/29-01-2026%2F4c56c0da_image.octet-stream)

I placed the LED right next to the red dot, and then made the USB port face the way of my laptop's internal USB board. I plan on physically soldering wires on my USB port from the inside and then hooking it up to my PCB. 

Now fast forward a few hours and this is what I ended up with

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Fa54ab445_image.octet-stream)

it's a very beautifully lay-ed out board and I did my best to route it nicely. I went with a 4 layer board with a SIG/GND/PWR/SIG stackup and did my best to route most of my signals on the top layer with a solid ground reference.

This is how I kind of plan to place it inside my laptop

![](https://cdn.2008000.xyz/cdn/29-01-2026%2F85875223_image.octet-stream)

## Time Spent: 3 Hours

## 21/01/26

Since I was done with the PCB I went over back to my schematic editor and assigned every component an LCSC ID (it was a pretty dreadful task). And once again I was lost in the JST naming convention and had a hard time finding the appropriate housing and crimp for the connectors.
But I kinda figured it out this time, I realized that for every JST series they have a part number of the housing, crimp and headers in the datasheet. 

After this I went back into the PCB editor and fixed up like a bajillion DRC errors, most of them were thrown because my components were too close to the edge, I changed up my DRC values to be more in line with my PCB fab's rules and most of my DRC errors went away, and for the remaining few I manually fixed them.

I went ahead and uploaded my component list and Gerber to get price quotes for my BOM. And after that I started working on setting up the repository for this project, added everything in a folder and wrote a README and then pushed it all to GitHub. While I was here I also took the time to download and assign 3D models to my footprints so I could get a cool screenshot of the render!

![](https://cdn.2008000.xyz/cdn/29-01-2026%2Fea0f258c_image.octet-stream)

## Time Spent: 1.5 Hours

