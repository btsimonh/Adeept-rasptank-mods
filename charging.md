# Adding charging to the rasptank:

I wanted the tank to be able to find a charge station and charge.

I also wanted to be able to manually charge using a 2S balance charger.

The first thing to do is to modify the battery cable to have a 3 pin socket on it, with the central pin being the connection between the two batteries.  
I created this cable such that it would plug directly int my balance charger balance port - however, I have subsequently found that my charger need to supply the main 
charge via a separate cable..  so will need to add a 2nd 2 pin socket for this.

For self charge, I purchased a simple 2S BMS - it has connections for the three battery connections and a P+/P- pair for input/output.  
Note that once connected to the batteries, this BMS needs to be 'woken' by connecting the charger to the P+/P- connections before it will output any voltage.

For charging, I'm not after a specifically fast charge, so chose a 1A 2S USB-C input charger.  
This combined with a magnetic USB-C connector and std USB wallwart will make manual charging simple, if a little slow.

This may also be 'just enough' to run the RPi3b for extended software development.  I can always upgrade it later...


## Charge power control.

For the case where the tank will find a charge station, and charge itself, I anticipate that the RPi will need to be powered down.

For this purpose I'm using a RPi Pico (although any micro with ADC, GPIs, GPOs, and low power modes would suffice, the pico was cheap).

To control the Rpi3b power, I've connected a flying lead to pin 4 of the power regulator on the Adeept motor control hat.  
The measured voltage on this pin is ~2v.  I did try a zener to gnd, and it seems that without a pullup, the zener passes enough current to turn off the regulator, 
so for the moment, I'm trusting that the voltage will not exceed the 3.3v that the Pico will take on a GPO, or that it won't supply sufficient current to damage the Pico.

So, the intention is that the Pico will sleep for extended periods, and wake to read the battery voltage, reporting it to the RPi3b.
If the voltage falls below 'regcharge' level, the tank software will move to the charging station and connect.

The Pico will be powered by a simple diode join of the 5v input to the USB charger and the 5v from the motor hat.
It will also have a 3.3v zener protected feed from each power source to GPI pins so it can know where it's power is coming from.
i.e. it can know if the charger is connected and supplying power.

Once the charger is connected, the RPi3b will tell the pico that it's about to shutdown, and then shutdown safely.
The Pico will wait long enough for shutdown to be complete, and then power down the motor hat.
It will then sleep for periods, monitoring battery voltage on wake.
Once the battery voltage has achieved 'charged' state, the pic will power up the motor hat.
On power on, the RPi3b tank software will determine if it is connected to a charge station, and of so, back away, so disconnecting itself, and then resume it's business.




