# ESPHome configs

All four pass `esphome config` on ESPHome 2026.10. Copy `secrets.example.yaml` to `secrets.yaml` (git-ignored), fill it in, and set the BMS's Bluetooth MAC in `bms-reader.yaml`.

| File | Device | Role | Needed? |
|---|---|---|---|
| [`bms-reader.yaml`](bms-reader.yaml) | ESP32-S3 | Reads the JBD BMS over BLE; **local low-battery and cold-charge failsafes** | Yes |
| [`bt-proxy.yaml`](bt-proxy.yaml) | M5Stack Atom Lite (any ESP32) | Passive Bluetooth proxy for the Victron MPPT | Yes, unless HA's own Bluetooth reaches the MPPT |
| [`vue-utility.yaml`](vue-utility.yaml) | Emporia Vue Utility Connect | Grid power from the smart meter, the controller's main input | Yes |
| [`s31-plug.yaml`](s31-plug.yaml) | Sonoff S31 | ~1 s power metering for one appliance | **Optional**, see below |

OpenDTU-OnBattery isn't ESPHome; flash it from its own [web installer](https://github.com/hoylabs/OpenDTU-OnBattery).

## Flashing
- **Vue Utility Connect:** follow [nekorevend/esphome-emporia-vue-utility](https://github.com/nekorevend/esphome-emporia-vue-utility). It's maintained upstream, so this repo doesn't copy the steps. You'll need a **3.3 V USB-UART adapter**, and your utility has to provision the device with your meter *before* you reflash it. Check meter/utility support before buying.
- **BMS reader / BT proxy:** plain USB, then OTA.
- **S31:** the same USB-UART adapter, after opening the plug (see below).

## About the Sonoff S31: a measuring tool, not a zero-export fix
The S31 is cheap (around $11) and gives ~1 s power readings, so the obvious idea is to put one on a big, fast-cycling appliance so the controller can react before the slower Vue does. **In practice that didn't meaningfully improve zero export here.** Three reasons:

- It only sees **one circuit**. The biggest export source in this house is the **AC compressor** switching off, and a plug can't meter a hardwired 240 V load.
- Most of an export event's size is set by how fast the **Hoymiles applies a new limit** (several seconds over the radio), not how fast the drop is detected. Even with a perfect sensor, the inverter keeps pushing until it catches up.
- The export from these events is already tiny, about 1 Wh per compressor shutoff, so there's little left to win.

What it **is** good for: measuring how big your fast-cycling loads are, and how often and how fast they swing, so you can tune the export buffer and step sizes from real data instead of guesses.

**Flashing is the hard part.** The case has to be pried open, and the programming points are flat test pads, not header pins. Without soldering or a clamp-style **pogo-pin jig** that presses pins onto the pads, holding a steady connection through the flash is fiddly. If you're building this project you'll already own the 3.3 V USB-UART adapter for the Vue, so the S31 adds no new tools. Just go in knowing it's for data gathering. **Never open or flash it while it's plugged into the wall.** Power it only from the adapter's 3.3 V.
