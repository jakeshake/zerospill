# Parts list

Prices are omitted on purpose because they change weekly. Bold items are the ones where the exact spec matters.

## Generation
| Part | Qty | Notes |
|---|---|---|
| 200 W panels | 6 | Wired 3S2P. **Check cold-weather Voc × 3 stays under the MPPT's 150 V limit.** |
| Tilt mounts (VEVOR 45" style) | 3 | Two panels per mount. Rate for your wind; secure against your worst wind direction. |
| MC4 2-to-1 Y branch connectors (PV side) | 1 pair (+/−) | Parallels the two 3-panel strings for 3S2P into the Victron. |
| **MC4 1-to-4 branch connectors (battery side)** | 1 pair (+/−) | Fans the battery feed out to **all four** HM-1200NT DC inputs. A 1-to-2 isn't enough; see the notes on the inverter row. Mind the gender trap ([build guide §3](build-guide.md#3-dc-protection)). |
| MC4 connectors + crimper | as needed | Includes the male-to-male / female-to-female adapter leads for the battery side. |
| PV wire (10 AWG) | as needed | |

## Charge + storage
| Part | Qty | Notes |
|---|---|---|
| **Victron SmartSolar MPPT 150/35** | 1 | BLE gets into HA via an ESPHome Bluetooth proxy. |
| **48 V (16S) LiFePO4, ~100 Ah** | 1 | Lossigy with JBD BMS (BLE). Any 16S pack with a readable BMS works. |

## DC protection + switching
| Part | Qty | Notes |
|---|---|---|
| **Fuse at battery positive** | 1 | 30 A MRBF used here. **Class-T recommended for the main position** (interrupt rating). Torque MRBF studs ≤ 75 in-lb. |
| **Pre-charge: 4-position rotary battery switch** | 1 | Rated 48 V DC, which is marginal for 16S (~58 V peak), so **only switch with no load**. |
| **25 Ω / 50 W wirewound resistor** | 1 | Stage 1 (B1–B2). |
| **0.2 Ω / 100 W wirewound resistor + momentary push-button** | 1 + 1 | Stage 2 (B2 → button → COM). Use a DC-rated button. See [build guide §4](build-guide.md#4-pre-charge-inrush). |
| **2-pole DC breaker, 30 A** | 1 | Directly before the inverter. |
| 10 AWG cable + ring lugs | as needed | Used throughout the DC side. |

## Inverter + AC
| Part | Qty | Notes |
|---|---|---|
| **Hoymiles HM-1200NT** (240 V US) | 1 | 4 DC inputs on 2 MPPT trackers, **11.5 A max per input**, 60 V max. At full output from a ~51 V bus it draws ~24–25 A DC, about 6 A per input across all four. Feeding only two inputs would push ~12 A each, over the per-input limit. |
| Hoymiles AC end cable (BC05 female) | 1 | Check your inverter's connector type. |
| SOOW 12 AWG cord | as needed | |
| **NEMA 6-20P plug** (or match your receptacle) | 1 | ≥ 20 A / 250 V. |
| Inline 2-pole breaker | 1 | Needed if the circuit's breaker is larger than the cord/pigtail rating. |

## Control + metering
| Part | Qty | Notes |
|---|---|---|
| **ESP32-S3 N16R8** (16 MB flash) | 1 | For OpenDTU-OnBattery. 16 MB is needed for OTA. |
| **NRF24L01+ PA+LNA** + breakout | 1 | Power from 3V3. Add a 10–47 µF electrolytic + 100 nF ceramic across VCC/GND. |
| ESP32 (ESPHome) for BMS + BT proxy | 1–2 | ESP32-S3 and/or M5Stack Atom. Splitting BMS reading and Victron proxying across two boards improved reliability. |
| **Emporia Vue Utility Connect** | 1 | Must be provisioned by your utility. **Only works with supported Zigbee smart meters**, so check before buying. Reflashed with ESPHome. |
| Sonoff S31 (optional) | 0–1 | **Measuring tool, not a zero-export fix.** ~1 s metering of one fast-cycling appliance. Fiddly to flash (open the case; pads, ideally a pogo-pin jig). [Details](../esphome/README.md#about-the-sonoff-s31-a-measuring-tool-not-a-zero-export-fix) |
| USB-UART adapter (3.3 V) | 1 | **Required** for flashing the Vue; also used for the S31. |
| IP65/67 enclosure | 1–2 | For the DTU and BMS boards if they live outdoors. |
| Multimeter | 1 | **Required.** Verify every polarity and receptacle voltage before energizing. |
