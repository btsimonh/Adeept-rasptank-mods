# Wheel sensor implementation

Although dead reckoning with tracks could be problematic, at least if we had wheel sensors, we could try.

One issue is that the amount of power to start the tracks is variable according to conditions, so
knowing if the wheels have started to move, and at what speed would be generally useful.

The approach i'm looking at is to print a black/white segments label for the wheel cogs.
Then using a pair of IR reflective sensors, we can detect wheel motion, and direction of wheel motion.
Note that thermal printed label 'black' is very IR reflective.  If you have this issue, cut away the black segments just leaving the black wheel behind.
Note also that the rear wheel's screws can collide with the motor mounting.  
I 'pruned down' The motor mounting corners to avoid this.  It's most evident when running the motors with no tracks....

Another optical issue is that the plastic of the tank is smooth and reflective.
I tested sanding the surface, but that is not necessarily the best solution.
One possible solution is to mount the IR at an angle, so that the reflected IR does not return to the IR receiver.  Or to use a matt black paint.

With sutiable interrupt driven GPI code, we should be able to track speed and distance, and so make motor drive PWM adapted to the desired motion.

After looking at a number of options, I found details about (pigpiod)[http://abyz.me.uk/rpi/pigpio/pigpiod.html]

It does almost exactly what I was wishing for - i.e. is able to read GPIs to high timing precision, and notify changes with timestamps - hence removing the need for real-time processing of the wheel sensor data.

Next was to find a nodejs client for pigpiod.
There are number, but many either use the C/C++ api, and need root access, or call 'pig' as a sub-process, or run python under the hood.  I needed something which would run as a normal user, and preferably had options for running on a remote PC.
After much research, I found (pigpio-client)[https://github.com/guymcswain/pigpio-client] which is a pure javascript implementation agains the sockets interface of pigpiod.
If provides timestamped monitoring of GPIs, and has a relatively simple implementation.

### Installing pigpiod
```
sudo apt-get update
sudo apt-get install pigpio
```
For me, this got v79 (RPi3b, bullseye).

The default configuration of the service only binds to ipV6, so you must change the service file.

edit  `/lib/systemd/system/pigpiod.service`

And replace `ExecStart=/usr/bin/pigpiod -l` with `ExecStart=/usr/bin/pigpiod`
`sudo systemctl daemon-reload`

*NOTE: this will enable pigpiod on ALL interfaces of the pi.  If you wish to have it bind ONLY to local IPv4, then use `ExecStart=/usr/bin/pigpiod -l -n 127.0.0.1`*

then start it
`sudo systemctl start pigpiod`

if it works for you, then:
`sudo systemctl enable pigpiod`



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


### Odometry calc:

see https://automaticaddison.com/how-to-publish-wheel-odometry-information-over-ros/




### Motor control via pigpiod

For controlling the motors, each has three GPOs.
One, connected to Enable, can be PWM to control the power to the motor.
The other two control the bridge direction.

I would like to control all three GPOs 'simultaneously'.  To do this, with pigpiod, it is possible to run a 'script' inside pigpiod.

*NOTE: pigpio-client does not currently support scripts.  I have put in a (PR)[https://github.com/guymcswain/pigpio-client/pull/39]...

For motor control, we setup the GPO ports using pigpio-client, then save a script to pigpio like:

`w p2 0 w p4 0 pwm p0 p1 w p2 p3 w p4 p5`

This script has 6 paramers, p0-p5.
p0 - pwm GPO number
p1 - pwm value (0-255)
p2 - bridge A GPO Pin
p3 - bridge A GPO value
p4 - bridge B GPO Pin
p5 - bridge B GPO value

The script writes 0 to A and B, then sets the pwm value, then writes new values to A and B

Example:
```
let scriptID = await pigpio.storeScript('w p2 0 w p4 0 pwm p0 p1 w p2 p3 w p4 p5');
await pigpio.runScript(scriptID, [17, 128, 18, 1, 27, 0]);
```



### References/other

Random links I found whilst researching

https://abyz.me.uk/rpi/pigpio/pigs.html describes the pipe interface.

https://flows.nodered.org/node/node-red-contrib-iiot-rpi-gpio for a node-red 'encoder' which will directly measure distance for us.  Problem is that it's based on non open C source, and so is not adaptable.

https://github.com/satoren/node-pigpio - a nodejs interface via sockets.  Incredibly complex code.

C examples - https://abyz.me.uk/rpi/pigpio/examples.html#pigpiod_if2%20code - should be adaptable.

pigpiod issues:
check:
sudo netstat -tulpn | grep pigpio
for where it's binding it's port.  if not ipv4, change:
see (this issue)[https://github.com/joan2937/pigpio/issues/203]

pigpio testing in nodejs:
`npm install --save pigpio-client`

index.js:
```
const pigpio = require('pigpio-client').pigpio({host: 'localhost'});

const ready = new Promise((resolve, reject) => {
  pigpio.once('connected', resolve);
  pigpio.once('error', reject);
});

ready
.then(async (info) => {
  // display information on pigpio and connection status
  console.log(JSON.stringify(info,null,2));

  // get events from a button on GPIO 17
  const clk = pigpio.gpio(10);
  const dt = pigpio.gpio(22);
  await clk.modeSet('input');
  await dt.modeSet('input');
  clk.notify((level, tick)=> {
    console.log(`clk changed to ${level} at ${tick} usec`)
  });
  dt.notify((level, tick)=> {
    console.log(`dt changed to ${level} at ${tick} usec`)
  });
})
.catch((e)=>{
  console.error(e);
});

```
