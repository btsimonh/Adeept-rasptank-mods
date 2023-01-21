# This is a repo to record my mods to the Adeept Rasptank

I have decided to develop the software for the robot from scratch using Node-Red.

This leads to some interesting investigations into how to use all of the sensors, motor control, and video from nodejs effectively.

## GPIO
For hardware control and sensing, I have found that the pigpiod demon on the raspberry pi is both highly performant, and pretty standard.  It allows interfacing via sockets, and so can be controlled/read from nodejs easily, although few have done it.  After all my investigations, this library (pigpio-client)[https://github.com/guymcswain/pigpio-client] interfaces with pigpiod well, and has all the features I could wish for.
The *main* feature pigpiod gives us over other GPIO methods is accurate timing of read data.  With nodejs being a little less than realtime, having timestamps is a godsend.
Another feature of pigpiod is that it can run 'scripts'.  This allows multiple GPOs to be controlled effectively simultaneously - so alleviating and node or network lag when controlling, for example, motors.  Information about scriptiing pigpiod is not necessarily easy to come by, but it's not hard in the end.
Yet another feature is that the control from pigpio-client can be done over LAN.  So we *could* split the robot brains/development out of the RPi and into a PC, with the robot becoming a dumb GPIO controller and video sender.

So far, I have tested wheel sensors and motor control.  There is a sample for pigpio for Ultrasonic rangefinding, and the line follow sensor is a no-brainer.

ToDo is servo control using I2C (through pigpiod?), and addressible LED control.

## Video
For video, I have video from the camera passing through OpenCV (local on the RPi), and arriving in browser using webRTC, with very low latency, and with hardware h264 encode, so low CPU overhead.
This is a combination of OpenCV/gstreamer and Janus, delivering to the Node-Red dashboard.

If we *wanted* to work with the video analysis being in a PC rather than in the RPi, then we'd need to somehow receive low latecny webRTC in OpenCV on the PC.


## Intended Modifications to the hardware of the robot itself:

## mod1 -  [Charging](./charging.md)

 add a cable to intercept the battery cable and allow charging from a balancing charger.
 add a battery management module.
 add a usb 2S charger
 add charge mode power control to power down the motor hat whilst charging.

## mod2 - [wheel sensors](./wheelsensors.md)

 add wheel sensors to sense position changes, and allow basic odometry.

