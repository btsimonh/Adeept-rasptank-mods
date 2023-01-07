# Wheel sensor implementation

Although dead reconing with tracks could be problematic, at least if we had wheel sensors, we could try.

One issue is that the amount of power to start the tracks is variable according to conditions, so
knowing if the wheels have started to move, and at what speed would be generally useful.

The approach i'm looking at is to print a black/white segments label for the wheel cogs.
Then using a pair of IR reflective sensors, we can detect wheel motion, and direction of wheel motion.

With sutiable interrupt driven GPI code, we should be able to track speed and distance, and so make motor drive PWM adapted to the desired motion.

The available IR sensors are rather large for the small robot, but one should fit internally above the motor, and one one the rear of the robot.  For the one on the rear, it would be fairly easy to make it's vertical position adjustable, so allowing adjustment of the phase of reading of the wheel segments.


The Adeept motor hat has two unused RGB sockets which present Vcc, gnd and two GPIOs from the RPi, and these can be used for the IR sensor digital outputs. 
