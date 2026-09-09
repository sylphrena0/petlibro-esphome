# PLWF105: Dockstream Smart Fountain

## Supported Features

- Turn pump on or off
- Red LED
- Yellow LED
- Scale (used to determine water level)
- Pump error indicator
- water consumption tracking
- water reporting in oz and ml
- configurable pump intervals
- integrated calibration process

## Disassembly

If you carefully open the water bowl "base" you will see a PCB holding a `ESP32-C3-WROOM-03`, a `HX7111` and a voltage regulator. There is also one more unidentified chip on the board

![PCB](https://github.com/user-attachments/assets/cf67e89f-4cc1-4773-8e06-78d1bb700e36)

## Connecting via Serial

The PCB very conveniently contains 3 pins that we can use to flash our firmware: `GND`, `TX`, and `RX`. The PCB also has a `VCC` pin that can be used to power the board while programming, but your USB-Serial converter needs to be able to supply enough current, so consider powering from the USB plug instead while programming. The PCB also has two pads on the lower right corner labeled `B` and `G`. These pads can be shorted to put the ESP chip in programmer mode.

You may be able to avoid soldering by holding wires to the contact points, but for a reliable connection you will likely be better off soldering contacts to these headers (ideally, contact points that can be disconnected from your wires).

## Entering Programming Mode

Do NOT power the board/fountain yet. Take a small wire and hold it on the `B` and `G` pads described above, then while still holding the wire in place, plug in the pump's USB cable. At this point the board should be in programming mode. If you left the top panel connected, the light should be a steady white. If it's slowly flashing white you did not enter programming mode.

If you run into problems or otherwise need to revert, you can reflash back to stock firmware using the backup you made in the "Dumping Stock Firmware" step of the top-level README.

## Result

Here is how it looks in HomeAssistant:

![Result](https://github.com/user-attachments/assets/24344f14-f331-4fa7-b6bd-9aeaddb32a11)

## Calibration

1) The water fountain will automatically enter calibration mode after a flash or after hitting "Start Calibration" in the home assistant UI
2) The fountain will now be in calibration mode (slow flashing yellow light).
3) With the water a the min fill line and fully assembled, hit the "wifi" button (or enter the number yourself in the home assistant UI)
4) The light should flash fast now, with the water a the max fill line and fully assembled, hit the "wifi" button (or enter the number yourself in the home assistant UI)
