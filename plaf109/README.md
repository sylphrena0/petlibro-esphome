# PLAF109: Polar Wet Food Feeder

## Supported Features

### On-Device Web Dashboard

This device has an optional custom UI that is enabled by default, source code at [sylphrena0/petlibro-esphome-plaf109-ui](https://github.com/sylphrena0/petlibro-esphome-plaf109-ui).

![Remote on a phone: feeding cards, chamber and configuration settings, and missed meal and battery warnings](https://raw.githubusercontent.com/sylphrena0/petlibro-esphome-plaf109-ui/refs/heads/main/docs/screenshots.png)

Note that screenshot uses mocked data (this feeder doesn't seem to get that cold).

### Functions

- reset button
  - restarts esphome when held for more than 2s
- automatic cooling to target temperature
  - option to manually disable cooling
  - automatically disables when power is lost to preserve power
  - NOTE: unsure about the utility of setting a target temperature
- feed now
  - plays chime
  - opens door, if not already open
  - waits for pet to arrive, up to a configurable timeout
    - optionally repeats chime at a configurable interval
    - closing the lid cancels the meal
  - if pet arrived
    - keeps door open for a configurable minimum feeding time after arrival
    - then waits till pet has been gone for 10s
    - closes door, if not manually closed
    - rotates plate
  - logs if pet arrived or not
  - closes door, if not already closed
- daily feeding schedule, stored on-device
  - feeds front plate, then rotates AFTER meal
  - uses same flow as manual feed, no special handling
  - warns when a meal is missed or cancelled (plate is not rotated in these cases)
    - warning survives reboots, is cleared at midnight, by a successful manual feed, or with the clear button
- open/close lid
  - will only close when pet is not present
  - using encoder to ensure lid opens/closes fully
- rotate plate
  - using encoder to ensure alignment
- presence sensor
  - daily visits and time at feeder
  - last visit time and duration
- status led
  - switch turns the white led on or off
  - pulses white while wifi is disconnected
  - pulses red while a missed meal warning is active, alternating red/white if wifi is also disconnected
  - warnings override the switch
- play chime

### Sensors

- battery voltage sensor
  - displays charge percentage, using factory formula
- ac power detection
  - may take several seconds to update after unplugged
- chamber temperature
  - not confident this is accurate, may need more calibration
  - uses factory formula

### To-Do

- use of motor current to detect stall
- maybe warn when a manual plate rotation offsets the feeding schedule
- scheduled meals will not trigger if set in close proximity (under 10-15m apart), maybe queue?
- better calibration of temp sensor, if possible
- bring some features to other devices
- support for feeding schedules for multiple days

## Disassembly

To access the motherboard, unscrew all visible screws on the bottom of the device. Remove all four feet and the battery cover, unscrewing those hidden screws as well. You do not need to unscrew the fan vent on the side.

![Image showing screws needed to disassemble device](./assets/base_screws.jpeg)

Carefully unclip the bottom cover. The battery, power, and reset cables will need to be disconnected to completely remove the cover. I recommend doing so and only reconnecting the power later for flashing, if you don't trust your serial adapter provides enough power to VCC.

When re-assembling, ensure all connectors are firmly in place. If you remove the motherboard, use caution disconnecting the headers. Mine were secured with hot-glue and very difficult to disconnect without damage.

*Please note that if you disassemble further for any reason: if you remove the cooling fins and TEC plate (ceramic sandwich under it), it is critical you put it back in the same orientation, or you may accidentally cook your food or even cause a housefire from heat buildup. You have been warned.*

## Connecting via Serial

The PLAF109 uses an [`ESP32-C3-WROOM-02U`](https://documentation.espressif.com/esp32-c3-wroom-02_datasheet_en.html#%5B12,%22XYZ%22,56.69,436.45,null%5D) and a `aw9523` I2C GPIO expander, and unlike the PLAF108 it has no exposed pads to short to easily enter programming mode.

![Image of motherboard showing chip](./assets/esp32_chip.png)

Connect the `3.3V`, `GND`, `TX`, and `RX` lines of your USB-to-serial adapter to the `VCC`, `GND`, `TX`, and `RX` headers on the motherboard.

Solder connections to these four connectors. You might be able to flash by manually holding contacts to them, but it'll be a pain. Refer to the image below:

![Image of motherboard showing VCC/GND/TX/RX connectors](./assets/motherboard.png)

![Pinout of the ESP32-C3-WROOM-02U, top of diagram is the side with the antenna connector](https://github.com/user-attachments/assets/a2bdcd5e-0cec-4a6d-9303-5f37277290ef)

## Entering Programming Mode

The ESP32-C3 starts in programming mode when `GPIO9` is pulled low (connected to `GND`) at power-on.

The contact pad labeled `TP4` on the back of the motherboard is a test point for `GPIO9`. To get access, carefully disconnect all wires and unscrew the motherboard. Use caution if your cables are secured with hot-glue, it is easy to break the connectors. In case you need to access GPIO9 after reinstalling the motherboard, solder a wire to TP4.

![Image of back of motherboard showing TP4](./assets/motherboard_back.png)

1. Disconnect all power sources from the motherboard.
2. Pull `TP4` (`GPIO9`) low by connecting it to `GND`. Connecting via a 1 kΩ resistor is optional but protects the pin.
3. Connect the USB-to-serial adapter to your computer.
4. Supply power to the motherboard. The chip starts in programming mode.
5. Disconnect `TP4` from `GND`.
6. Backup and flash, continuing guide from the [main README](../README.md#flashing-esphome).
7. Disconnect and reconnect power. The chip will load the new firmware.

If the ESP32-C3 does not start in programming mode, make sure that the wire between `TP4` and `GND` has a good connection. Then repeat steps 1 to 4.

If your serial adapter has `DTR` and `RTS` lines, you can let `esptool` enter and exit programming mode automatically instead of power-cycling by hand. Connect `DTR` to `GPIO9` (`TP4`) and `RTS` to `EN`, then connect or power the board so `esptool` pulls `GPIO9` low. Use a 1 kΩ series resistor on `DTR` to protect the pin, or short it directly if you live life on the edge. Per the chip manual you may only need `DTR` to `GPIO9`, but connecting `RTS` to `EN` also works and lets `esptool` reset the board after flashing.

## Customization

You can customize the configuration with substitutions:

```yaml
substitutions:
  name: plaf109  # custom name for your device
  friendly_name: Polar Wet Food Feeder  # used in HAOS, dashboard, and default UI
  dashboard: true  # enable custom dashboard, accessible via <device-ip>/dashboard
  dashboard_path: /dashboard  # only required if you want a path that isn't /dashboard

packages:
  plaf109: github://sylphrena0/petlibro-esphome/plaf109/config.yaml@main
```

If that's not enough, you can override components manually. If you wish to not require auth for the dashboard/web_server, for example:

```yaml
packages:
  plaf109: github://sylphrena0/petlibro-esphome/plaf109/config.yaml@main

web_server:
  auth: !remove
```

Or if you wish to save flash space and not include the dashboard at all:

```yaml
substitutions:
  dashboard: false

packages:
  plaf109: github://sylphrena0/petlibro-esphome/plaf109/config.yaml@main
```

## Data Provenance

In order to gain feature parity with the stock app, I used a combination of dumping and reverse engineering the factory firmware and I2C sniffing (see [main README](../README.md#adding-a-new-device)).
