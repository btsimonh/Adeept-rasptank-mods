# Wheel sensor implementation

Although dead reconing with tracks could be problematic, at least if we had wheel sensors, we could try.

One issue is that the amount of power to start the tracks is variable according to conditions, so
knowing if the wheels have started to move, and at what speed would be generally useful.

The approach i'm looking at is to print a black/white segments label for the wheel cogs.
Then using a pair of IR reflective sensors, we can detect wheel motion, and direction of wheel motion.
Note that thermal printed label 'black' is very IR reflective.  If you have this issue, cut away the black segments just leaving the black wheel behind.
Note also that the rear wheel's screws can collide with the motor mounting.  
I 'pruned down' The motor mounting corners to avoid this.  It's most evident when running the motors with no tracks....

With sutiable interrupt driven GPI code, we should be able to track speed and distance, and so make motor drive PWM adapted to the desired motion.
(see https://flows.nodered.org/node/node-red-contrib-iiot-rpi-gpio for a node-red 'encoder' which will directly measure distance for us).

The available IR sensors are rather large for the small robot, but one should fit internally above the motor, and one one the rear of the robot.  For the one on the rear, it would be fairly easy to make it's vertical position adjustable, so allowing adjustment of the phase of reading of the wheel segments.


The Adeept motor hat has two unused RGB sockets which present +3.3v and three GPIOs (pulled up with 10k) from the RPi each, and these can be used for the IR sensor digital outputs. 
We need to find a GND....

RGB1 has:
+3.3v
GPIO022(GPIO_GEN3)
GPIO23 (GPIO_GEN4)
GPIO24 (GPIO_GEN5)

RGB2 has:
+3.3v
GPIO10 (SPI_MOSI)
GPIO09 (SPI_MISO)
GPIO25 (GPIO_GEN6)



Node-red:
https://flows.nodered.org/node/node-red-contrib-iiot-rpi-gpio
Deals with a rotary encoder - so if we have two sensors per wheel, we will have position.
For the rotary encoder, it needs a 'button' GPI.  The motor hat we are using has two 'spare' gpis that we are not using...

Servos:
https://flows.nodered.org/node/node-red-contrib-iiot-rpi-pca9685


