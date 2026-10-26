# Troubleshooting (real problems from this build)

## Main fuse blew instantly on first power-up
**Symptom:** A 30 A MRBF at the battery terminal blew with a flash while rotating the pre-charge switch to position 2, *before* the inverter's DC breaker was closed.

**Why a bigger fuse isn't the answer:** With the breaker open there's no path to the inverter, so inrush can't be the cause. An instant blow points to either:
- a **short in the switch/resistor wiring** (a stacked lug touching the housing or a mounting screw, or a conductor on the wrong stud), or
- a **cracked or over-torqued MRBF**. The spec is ≤ 75 in-lb. Over-torquing cracks the blade and causes arcing at the joint.

**Fix:** With everything de-energized, continuity-test B1/B2/COM to battery − and to chassis. Re-verify each stud against the diagram, replace the fuse, torque to spec, and then follow the pre-charge sequence exactly.

## DC polarity into the inverter is backwards
**Cause:** MC4 Y-splitters are made for combining panels. Feeding one battery lead *out* through them to several inverter inputs reverses which gender carries which polarity.

**Fix:** Male-to-male and female-to-female MC4 adapter leads between the DC breaker and the splitters, so + lands on the inverter's + inputs. Meter the inverter-side plugs before every first connection. See [build guide §3](build-guide.md#3-dc-protection).

## OpenDTU-OnBattery can't see the NRF24
- IRQ pin left at −1 → set it.
- Radio wired to a 5V0 pin that isn't populated on the board → power from 3V3.
- Still nothing? Flash plain OpenDTU on a spare board with the same radio. If that works, suspect a defective board.

## Packet loss / inverter "unreachable"
OpenDTU radio stats showed ~42 % RX success, 58 % partial receives, and −80 dBm RSSI.
- Add decoupling caps at the radio.
- **Move the DTU away from the Wi-Fi router.** This was the bigger factor here.
- Shorten the distance and clear the line of sight to the inverter.

## Brief export spikes
Mostly caused by **AC compressor and air fryer cycling**: the load drops faster than the Vue reports.
- The controller cuts fast and raises slowly.
- A dedicated fast-cut automation acts on ~1 s readings from an S31 on the air fryer.
- Keep a small import buffer (~30–50 W) and a guaranteed base load (fridge/freezer) as the floor.

## Battery charging in the cold
The battery is outdoors. The BMS ESP32 enforces a cold-charge lockout locally, so it doesn't depend on HA being up.
