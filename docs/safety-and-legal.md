# Safety & Legal — read this first

I'm a homeowner sharing what I built, not an electrician or a lawyer. This page sets out the facts so you can decide.

## California SB 868 (Plug and Play Solar Act) and similar laws

SB 868 was signed Sept 30, 2026 and takes effect Jan 1, 2027. Unless extended, its exemption sunsets in 2030. Laws in other states (Utah, New Jersey, and others) follow the same pattern. In broad strokes, they exempt **plug-in** solar from utility interconnection when the device:

- connects through a **standard 120 V receptacle**
- is capped at **1,200 W AC**
- is **certified to UL 3700** (or an equivalent) as a plug-in PV system

**This build does not meet those conditions:**

| Requirement | This build |
|---|---|
| 120 V standard outlet | ❌ 240 V receptacle on a dedicated circuit |
| UL 3700 listed system | ❌ DIY assembly of individually listed parts |
| Plain PV-to-inverter | ❌ Battery-fed inverter on a DIY DC bus |

So **don't treat this as "legal under the new law."** Running it is a grid-connected inverter installation, and in principle that means local permitting and utility interconnection rules apply. Zero-export control reduces grid impact, but it doesn't change the legal category. Check with your utility and your local building department.

Sources: [KPBS](https://www.kpbs.org/news/quality-of-life/2026/10/01/california-formally-legalizes-smaller-cheaper-balcony-solar-panels), [pv magazine USA](https://pv-magazine-usa.com/2026/10/01/newsom-signs-california-plug-in-solar-bill-establishing-the-nations-largest-balcony-solar-market/), [UL plug-in PV safety considerations](https://www.ul.com/insights/safety-considerations-plug-photovoltaic-pipv-systems)

## The real hazards

1. **Cord-connected backfeed.** When the inverter is producing, the prongs of its plug are live once unplugged until anti-islanding shuts it down (Hoymiles does this quickly, but don't rely on it). Never handle the plug with the inverter producing. Turn off the DC breaker first. A hardwired connection by an electrician is the safer choice.
2. **Circuit and conductor protection.** The inverter's AC pigtail is light gauge. If your 240 V circuit is on a big breaker (an old 50 A spa circuit, for example), add an appropriately sized 2-pole breaker inline so the small conductors are protected.
3. **Lithium battery fault current.** A 48 V LiFePO4 pack can deliver thousands of amps into a short. Fuse at the battery terminal, and consider Class-T for the main fuse. Torque fuse studs to spec.
4. **Inrush.** The inverter's input capacitors cause a large spark and current spike on connection. Pre-charge them (see build guide).
5. **Input voltage.** A 16S LiFePO4 pack at full charge sits close to the Hoymiles HM-series max DC input (60 V; check your datasheet). Keep charge voltages conservative.
6. **Manufacturer intent.** Hoymiles microinverters are designed for PV inputs. Feeding one from a battery is common in the OpenDTU-OnBattery community but outside the manufacturer's intended use, and it may affect your warranty.
7. **Cold charging.** LiFePO4 must not be charged below freezing. Use BMS protection and/or a software lockout.
