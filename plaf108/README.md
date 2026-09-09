# PLAF108: Air Smart Feeder

## Supported Features

- Alarm Red LED
- Sensor for showing if feeder is on battery power
- Sensor for showing if battery is plugged in
- Sensor to enable the "Food Detection" sensor
- Sensor for showing if mains connected
- Switch to turn motor left
- Switch to turn motor right
- Sensor to show when the motor has done a quarter turn

GPIO0 and GPIO1 are unidentified. I think it's for the DC-power current sensor to maybe determine battery charge.

## Disassembly

*Request for contributions: instructions on disassembly.*

## Connecting via Serial

The PCB very conveniently contains 4 pins that we can use to flash our firmware. `GND`, `TX`, `RX`, and `VCC`. The PCB also has two unlabeled pin holes. These holes can be connected to each other put the ESP chip in programmer mode.

You may be able to avoid soldering by holding wires to the contact points, but for a reliable connection you will likely be better off soldering contacts to these headers (ideally, contact points that can be disconnected from your wires).

## Entering Programming Mode

Take a small wire and hold it in the two unmarked pin holes described above, then while still holding the wire in place, plug in the board. At this point the board should be in programming mode.

*Request for contributions: a picture of the contact points mentioned would make these instructions much clearer.*
