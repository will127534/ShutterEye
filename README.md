# ShutterEye

A basic minimum **SC910GS** (1" / 3840x2160 / Global Shutter) board for Raspberry Pi.

<img width="1280" alt="image" src="https://github.com/user-attachments/assets/1c6bf67c-d5cb-4472-8dc6-60813cfa6754" />

In general this is a simple breakout board for the SC910GS sensor — see the driver repo [here](https://github.com/will127534/sc910gs-v4l2-driver).


## Repo layout

- `Gerbers/` — manufacturing files for JLCPCB including CPL files
- `bom/` — BOM / parts list [Interactive BOM](https://htmlpreview.github.io/?https://github.com/will127534/StarlightEye/blob/main/bom/ibom.html) [(provided by InteractiveHtmlBom)
](https://github.com/openscopeproject/InteractiveHtmlBom)


## Images

<img width="1280" alt="image" src="https://github.com/user-attachments/assets/49d68a3f-a30e-4fb1-a190-da6f73e8cc9e" />

<img width="1280" alt="image" src="https://github.com/will127534/ShutterEye/blob/3ecd99c01cc8732a6afa812ae9d3f57335682a9c/pcb.jpg" />
<img width="1280" alt="image" src="https://github.com/will127534/ShutterEye/blob/3ecd99c01cc8732a6afa812ae9d3f57335682a9c/sch.jpg" />


## Usage Notes

Driver repo: [https://github.com/will127534/sc910gs-v4l2-driver](https://github.com/will127534/sc910gs-v4l2-driver)  
Libcamera fork that supports this sensor: [https://github.com/will127534/libcamera/](https://github.com/will127534/libcamera/)

Basically you can follow the quick start guide for StarlightEye/OneInchEye/FourthirdEye, just swap the driver link to sc910gs and dtoverlay to `dtoverlay=sc910gs`, libcamera are the same across sensors.

Also note that the camera calibration and tuning files are done with IR650 cut filter.

## Random(Yapping) Footnotes

Yes I ran out of naming ideas because this sensor is essentially same as the GlobalEye (GMAX3412 1" 4K 30p global shutter), so I have to name it to ShutterEye instead. As for the difference: SC910GS is much suited for Single Board Computers comparing to GMAX3412 beacuse no external exposure signals needed to trigger new frames. Another differences is that compare to GMAX3412 even though they are both ~1" 4K 30fps global shutter sensor, GMAX3412 has a larger 4096 x 3072 image compare to SC910GS at 3840x2160.  

On the right is GMAX3412, On the left is SC910GS, you can see it is a bit narrower (16:9 vs 4:3)
![_DSC1365](https://github.com/user-attachments/assets/b48bea69-9c61-43fe-ac82-e3e3d8568ead)

Now going back to SC910GS itself, the datasheet I'm manage to get without NDAs are quite.... sparse interms of the details, specifically framerate control, and I have to cross reference other smartsense image sensor to get the framerate control working. However, one of the thing I want to enable is the trigger signal output so I can put 6-axis IMU on the board similar to all the image sensor boards I've designed, but I just can't get it to work so I end up skipping it. (Also ICM42688-P is out of stock everywhere for some reason, help).

On the HW side of things, lets just say that I don't understand how manufacture can suggest a almost impossible amount of bypass capacitors in the reference design, like did you(The one who draw such sch) even think about layout? There is no F-ing way that I can put 10uF + 0.1uF on EVEY SINGLE POWER PINS, as the resut in my sch you will see that I shrink it down to single 1uF each.   
<img width="1034" height="862" alt="image" src="https://github.com/user-attachments/assets/0bb31742-6ba7-4751-a701-c672b2a41b1e" />

Another issue rise with such a capacitor array is the in-rush current when powered on, specifically Raspberry Pi 5 has a very sensitive overcurrent detection, which can be a good thing for short circuit detection but in this case in my beta version of the board it trips that OC detection with that amount of capacitors when all the power rails turned on at once by the enable pin. So the solution I have to implment is soft start for all the power rails other then VDDIO, and VDDA and VDDP has the largest soft start time delay. Now I haven't even finish yapping about the capacitors yet because there are still charge pump capacitors as global shutter image sensor tends to require more power and more power rails, one of the issue(?) is that a bunch of rails like VRM/VRSFLO... have like two pins sepperated on differnt side of the chip! and I need to bridge them together. Because of the pin density and layout reasons it is not a large power plane but just a few narrow traces connect them together, not ideal but it seems to be working just fine.

As for the OSC frequency I have to go with 27MHz specifically because the datasheet I have didn't even bother documenting the PLL configuration, and the only thing I have is the initializing register list that labeled for 27MHz. Finally, continuing the lack of documentations, I have the L0 ~ L3 + G0,G1 LED strobe and GPIO breakout to test points but I have no idea how to get it to work (I tried).

On the driver coding side of things it is done quite easily, what I really like is the sensor's quite high analog gain, which is handy specifically for global shutter sensor where you need very quick exposure speed but you've maxed out the lighting you have, and having the capability to just crank up the gain is nice, though yes the noise too.

Other then the lack of some critical details, most of the (normal) sensor functions should works, haven't tried the HDR features yet but there seems to be some trick up its sleeves with outputing different gain readout as two lines, so I guess you can kinda treat is as quad bayer while in the RAW data collection you can do your own HDR while the preview maybe still useable.

Overall I hate it when placing the capacitors, I hate it when working on the layout, I hate it when the framerate control and PLL config is not even in the datasheet that I have to cross reference and just buy new 27Mhz OSCs, but the end result is quite nice and driver mostly works and having high gain on global shutter do make the actuall imaging use simplier.
