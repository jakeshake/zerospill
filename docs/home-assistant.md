# Home Assistant

## How the controller works
Every cycle, the controller:
1. Computes **house load** as raw grid power (Vue) **plus** inverter AC output, sampled at the same moment.
2. Sets a **target**: house load minus a small import buffer (~30–50 W), capped at the inverter's rated power and by battery SoC / time-of-use rules.
3. **Cuts fast** when the grid goes negative (export), and **raises slowly and in capped steps** when import grows.
4. Turns the inverter off (gated, not on every dip) when the target stays below a useful minimum.

Supporting pieces:
- **Optional fast-cut automation:** drops the limit as soon as a ~1 s plug meter (Sonoff S31) sees a big appliance turn off. It isn't in the package because the gain was small. Inverter response time, not sensing, sets the size of an export spike.
- **Limit-correction learner:** learns the ratio between the commanded limit and actual output.
- **Overnight hold / forecast:** uses Solcast to decide how much battery to reserve for the 5–8 pm peak and overnight.
- **Alerts:** stale data (DTU unreachable), grid export, low-yield day, high wind for the panels.

## Install

### Option A: package (recommended)
A blueprint can only create an automation; it can't create the helpers (`input_number`, template sensors, utility meters) this system depends on. The package creates all of them.

1. Enable packages in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
2. Copy [`home-assistant/packages/zero_export_solar.yaml`](../home-assistant/packages/) into `config/packages/`.
3. Search/replace the placeholder entity IDs listed at the top of the file with yours (grid sensor, OpenDTU limit/power/producing/buttons, BMS SoC/voltage/charge switch, Solcast, notify service).
4. Restart HA, then run **Zero-Export Solar - Set Default Values** once (Settings → Scripts). The helpers deliberately have no `initial:` value, because that would reset your tuned and learned values on every restart.
5. Set **Solar Peak Power** for *your* DC side. It's the hard ceiling in every mode. Here it's 1,150 W, which keeps DC current at or under ~24 A (80 % of a 30 A breaker/fuse on 10 AWG).
6. Before trusting auto mode, test with **Hoymiles Manual Control** on and a small manual limit, and confirm the inverter follows.

What's in the package:

| Piece | Purpose |
|---|---|
| `sensor.grid_power_median` | 3-reading median, so single-sample dips are ignored |
| `sensor.house_load` | Grid + inverter output paired at the same moment; holds during limit changes |
| `sensor.solar_safe_load` | House load minus the export buffer (falls back to base load if the meter is stale) |
| `sensor.solar_early_release` | Daytime discharge the battery can afford and still hit the target SoC by 17:00 (Solcast P10) |
| `sensor.solar_dispatch_target` / `_mode` | What to deliver now: peak pacing, overnight hold, daytime release, floors |
| Controller + learner | Limit control and commanded-vs-actual correction |
| Overnight hold, daytime backstop | Forecast-sized reserve; full-battery overflow |
| Alerts | Sustained export / >6 Wh per 10 min; stale data; unreachable inverter |
| Battery charge cap (optional) | 95 % cap with monthly top-balance, via the BMS charge switch |

The 5–8 pm peak window, battery capacity and minimum voltage are all helpers, so no template edits are needed for a different tariff or pack.

### Option B: blueprint (controller only)
For people who already have their own sensors and helpers. Import [`home-assistant/blueprints/zero_export_controller.yaml`](../home-assistant/blueprints/) and pick your entities in the UI.
<!-- TODO: publish to the HA Community "Blueprints Exchange" and add the import badge here -->

## Dashboard
![Home Energy dashboard, week view](../images/home-energy-week.jpg)

The Home Energy dashboard is a standalone HTML page served from `config/www/` and added as a Webpage dashboard. Source and install steps: [`home-assistant/dashboard/`](../home-assistant/dashboard/). It reads your Energy dashboard settings, so the only edits are your battery SoC and inverter entities and your peak hours.
