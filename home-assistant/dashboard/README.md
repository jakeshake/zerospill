# Home Energy dashboard

A single-file, standalone energy page for Home Assistant. It has four tabs: **Now** (live power flow + current price), **Today** (5-minute chart, battery SoC, where your power came from, cost and savings), **Week** (7-day breakdown) and **System** (every source sensor and how fresh it is). There's no build step, add-on or HACS install.

![Week view](../../images/home-energy-week.jpg)

## How it works
- It reads your solar, battery, grid and price entities from **Settings → Dashboards → Energy**, so set those up first. It works with both the current and the pre-2025 grid formats.
- It connects to HA's WebSocket API from your browser and uses long-term statistics, so the history comes from the recorder. No extra sensors are needed.
- Savings are costed hour by hour at your live price entity: energy that came from solar or the battery × what that hour would have cost from the grid.

## Install
1. Copy `energy.html` to `/config/www/energy.html`. Create `www` if it doesn't exist, and restart HA the first time you create it.
2. Edit the `CONFIG` block near the top of the script:
   - `socEntity`: your battery state-of-charge sensor (**required**)
   - `inverterEntity`: inverter AC output (recommended for battery-fed microinverters), or `null`
   - `dischargeEntity`: BMS discharge-FET binary sensor, or `null`
   - `peakHours`: your on-peak window, e.g. `[17, 20]` for 5–8 pm
3. Add it to the sidebar: **Settings → Dashboards → Add dashboard → Webpage**, URL `/local/energy.html`.

## Sign-in
In the HA sidebar or the same browser, it reuses your existing Home Assistant login (with "Keep me logged in" checked). On a device without a session, it shows a box where you can paste a **long-lived access token** (Profile → Security). That token is saved only in that browser's local storage and is never written into the file. Don't hard-code a token into `energy.html`, because everything in `/config/www` is served without authentication.

## Credits
Visual idea inspired by u/Potential-Cod-1851's "Home energy" page. Code written from scratch.
