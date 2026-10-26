# Build guide

Order matters. Don't connect the inverter to the battery until sections 1–5 check out with a multimeter.

## 1. Panels
- 6 × 200 W in **3S2P**: two strings of three in series, paralleled with MC4 Y connectors.
- Check **cold Voc**: panel Voc × 3 × a cold-temperature factor (~1.1) must stay under 150 V for the Victron 150/35.
- South-facing at ~20° tilt. Leave enough row spacing that the rows don't shade each other at your worst sun angle.

## 2. Charge controller + battery
- Victron SmartSolar MPPT 150/35 → 48 V LiFePO4.
- Set charge voltages in the **VictronConnect app** (HA can't change absorption).
- **Keep absorption conservative.** The Hoymiles HM-series DC input max is 60 V, and a 16S pack at 100 % is ~58.4 V. Running 5–95 % SoC also extends cycle life.

## 3. DC protection
Battery + → **fuse at terminal** → pre-charge rotary switch → **30 A 2-pole DC breaker** → inverter inputs. 10 AWG throughout. Battery − returns directly.

- At 1,200 W from a ~51 V bus, expect ~25 A DC (more as voltage sags). Confirm fuse, wire, breaker and BMS discharge rating.
- The battery bus must feed **all** inverter inputs (MC4 Y splits), because each input has a per-input current limit.
- **⚠️ MC4 polarity: the Y-splitter trap.** MC4 genders are a polarity *convention*. Panel leads and inverter inputs are gendered so that + can only mate with +. Y-branch connectors are built to **combine panels into one inverter input**. This build runs power the other way: **one battery feed fans out to several inverter inputs**. With power flowing backwards through the splitters, the connector genders end up opposite to what the polarity convention expects. If you crimp the breaker-side leads the "normal" way, the plugs either won't mate or, worse, will deliver **reversed polarity into the inverter.**
  - **Fix used here:** build short **male-to-male and female-to-female MC4 adapter leads** between the DC breaker outputs and the splitters. That way, at the inverter end, + arrives on the inverter's + input and − on its − input.
  - Mark every lead with red/black heat-shrink. Gender alone can't be trusted once adapters are in the chain.
  - **Meter every connector that will plug into the inverter before mating it**: + on the inverter's + input, − on −, ~48–58 V. Reversed DC on a microinverter input can destroy it.

## 4. Pre-charge (inrush)
The inverter's input capacitors look like a dead short for a few milliseconds. Hard-connecting them to a lithium pack causes a big spark, pitted contacts, and can pop fuses.

**Wiring on a 4-position marine rotary switch (OFF / 1 / 1+2 / 2):**
- **B1:** one lead of a 25 Ω / 50 W resistor
- **B2:** battery + (via fuse), stacked with the other resistor lead
- **COM:** to the 30 A DC breaker → inverter

**Sequence:**
1. AC unplugged, DC breaker **open**.
2. Switch OFF → **1** (through the resistor). Hold 3–5 s.
3. Turn straight to **2**. Don't park at 1+2.
4. Close the DC breaker, then plug in AC.

To shut down, reverse the order: unplug AC, open the breaker, switch OFF. **Never rotate the switch under load.** Its DC rating is marginal for 16S.

## 5. AC connection
**Verify the receptacle first** (circuit off for wiring, on for measuring):
- L1–L2 ≈ 240 V; each hot to ground ≈ 120 V
- Ground is bonded and continuous back to the panel
- The circuit is **dedicated** (nothing else on it) — confirm by switching the breaker off and checking what dies
- Old spa/dryer receptacles (e.g. NEMA 10-50) may carry **no true ground**, or a ground landed on the neutral pin. Fix this properly (e.g. a 6-50R with a real ground) before using it.

**Inverter cord (HM-1200NT, 240 V, no neutral):**
| Hoymiles AC conductor | SOOW | 6-20P |
|---|---|---|
| Brown (L1) | Black | Brass terminal |
| Blue (L2) | White, **re-identified as hot with tape** | Other brass terminal |
| Green/yellow (PE) | Green | Green ground screw |

The two hots are interchangeable on 240 V split-phase. What matters is that ground goes to ground and no hot lands on a ground or neutral. Trust the L/N/PE markings molded into your connector over wire colors, which vary by revision.

If the circuit breaker is larger than the cord/pigtail rating, add an inline 2-pole breaker sized for the smallest conductor.

## 6. DTU (OpenDTU-OnBattery)
- ESP32-S3 N16R8 + NRF24L01+ PA+LNA. Flash [OpenDTU-OnBattery](https://github.com/hoylabs/OpenDTU-OnBattery) via its web installer.
- **Pin config:** set the IRQ pin (don't leave it at −1). Power the radio from **3V3**. Some dev boards have an unpopulated 5V0 pin.
- Add **decoupling caps** at the radio: 10–47 µF electrolytic (mind polarity) + 100 nF ceramic across VCC/GND. Lever-nut connectors work if you don't solder.
- **RF placement matters more than you think.** Keep it away from your 2.4 GHz Wi-Fi router, with a clear path to the inverter. Check OpenDTU's radio stats (RX success rate, RSSI).
- Add the inverter by serial and enable night polling/commands so the controller can manage it off-battery.
- Publish to MQTT and enable HA auto-discovery.

## 7. BMS + Bluetooth proxy
- An ESP32 running ESPHome reads the JBD BMS over BLE and also acts as a Bluetooth proxy for the Victron. See [`esphome/`](../esphome/).
- If BLE gets flaky, split the roles across two boards (BMS reader / BT proxy).
- **Local failsafes on the BMS ESP32**, which keep working without HA:
  - turn off the discharge MOSFET at a low-SoC floor
  - block charging below a cold threshold

## 8. Grid metering
- **Emporia Vue Utility Connect** reads your smart meter over Zigbee. It must be **provisioned by your utility**, and it **only works with supported meters/utilities.** Check before buying.
- Flash it with ESPHome using **[nekorevend/esphome-emporia-vue-utility](https://github.com/nekorevend/esphome-emporia-vue-utility)**. Follow that repo's instructions; they're maintained upstream.
- The Vue updates every few seconds to ~30 s depending on the meter, and that's the controller's sensing bottleneck. An ESPHome-flashed Sonoff S31 gives ~1 s readings for one appliance. It's useful for *measuring* fast-cycling loads, but it didn't meaningfully improve zero export here (see [esphome/README](../esphome/README.md#about-the-sonoff-s31-a-measuring-tool-not-a-zero-export-fix)).

## 9. Home Assistant
See [home-assistant.md](home-assistant.md).
