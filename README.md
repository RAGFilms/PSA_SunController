# PSA_SunController
Sun controller for Physical Starlight Atmosphere, by Physical Addons, that allows you to set the atmosphere by time, date, season and geographic location.
# PSA Sun Control Extension v1.0
**Real-World Solar Positioning for Physical Starlight & Atmosphere**
*by AGW Entertainment*

Compatible with **Blender 5.1+**

---

## Overview

PSA Sun Control Extension adds timezone-aware, astronomically accurate solar positioning to Blender's **Physical Starlight and Atmosphere** (PSA) addon. Instead of manually dialing in sun angles, you enter a real-world location, date, and time — the addon calculates the correct azimuth and elevation using the NOAA/Astronomical Almanac algorithm and pushes the result directly into your PSA node.

No external Python libraries required. All math runs inside Blender.

**What's included:**
- Real-world solar position calculation (NOAA algorithm with atmospheric refraction correction)
- Timezone selector covering all major world zones
- Live Update mode — PSA updates as you move the time slider
- 13 built-in lighting presets organized by category (Golden Hour, Sunrise, Sunset, Blue Hour, etc.)
- Three-stage PSA node detection that works with multiple PSA versions
- Driver node injection that correctly overrides linked PSA sockets
- One-click restore to return PSA to manual control

---

## Requirements

- Blender 5.1 or newer
- **Physical Starlight and Atmosphere** addon installed and active in your scene
  - PSA must be applied to the World — the World node tree must contain a PSA group node

> PSA Sun Control is a companion tool for PSA. It does not include or replace PSA itself.

---

## Installation

1. Download `psa_sun_control_v1.0.zip` from the [Releases](../../releases) page
2. Open Blender and go to **Edit → Preferences → Add-ons**
3. Click **Install...** and select the downloaded zip file
4. Find **PSA Sun Control Extension** in the list and enable the checkbox
5. Open the **Properties panel → World Properties** — the PSA Sun Control panel appears there

---

## Basic Workflow

```
Set Location & Time → Scan World Nodes → Select PSA Node → Calculate & Apply
```

**Step 1 — Set your location and time**
Enter your latitude, longitude, timezone, date, and time of day in the panel. You can also load a built-in preset from the Presets sub-panel to start from a known reference point.

**Step 2 — Scan World Nodes**
Click **Scan World Nodes**. The addon scans your World node tree for group nodes and lists them in the node selector dropdown. The System Console prints a full socket breakdown for each node found — useful for diagnosing unusual PSA setups.

**Step 3 — Select your PSA node**
Pick your PSA group node from the dropdown. Nodes with recognized sun sockets are automatically flagged and pre-selected.

**Step 4 — Calculate & Apply**
Click **Calculate & Apply**. The computed azimuth and elevation appear below the button. The sun position is pushed into PSA immediately.

---

## Panel Reference

### Status Indicator
The top of the panel shows whether a PSA node is currently connected. If it reads **PSA Node Not Found**, run Scan World Nodes and select your node.

### PSA Node Section

| Control | Description |
|---|---|
| Scan World Nodes | Scans the active World node tree and populates the node selector |
| Node Selector | Dropdown of all group nodes found in the scan. Pick your PSA node here. |
| Restore PSA Controls | Removes the driver nodes injected by this addon, returning PSA to its default manual control |

### Live Update
When enabled, PSA updates automatically every time you change any setting — timezone, date, time, or location. Toggle it off if you want to batch-adjust settings before applying.

### Timezone
Select your timezone from the dropdown. Covers all major world zones including DST variants (EST/EDT, CST/CDT, PST/PDT, etc.) plus Europe, Africa, Middle East, Asia, and Oceania.

### Date
Set the **Year**, **Month**, and **Day** for the solar calculation. The algorithm is accurate from 1900 to 2100.

### Time of Day
A slider from 0.0 (midnight) to 23.99 (11:59 PM). The current time displays above the slider in both 24-hour and 12-hour formats.

### Location
| Field | Description |
|---|---|
| Latitude | Decimal degrees. Positive = North, Negative = South. Range: –90 to +90. |
| Longitude | Decimal degrees. Positive = East, Negative = West. Range: –180 to +180. |

### Calculate & Apply
Runs the solar position calculation and pushes the result to the selected PSA node. The computed azimuth (degrees clockwise from North) and elevation (degrees above horizon) display below the button. A warning appears if the sun is below the horizon for the given settings.

---

## Presets

The **PSA Presets** sub-panel provides 13 built-in location/time references organized by lighting category. Click any preset to load it — if Live Update is on, PSA updates immediately.

| Category | Presets Included |
|---|---|
| Golden Hour | Los Angeles (PST, Nov), New York (EST, Oct) |
| High Noon | Sahara (June solstice), Equator (June solstice) |
| Sunrise | Tokyo (March equinox), Mojave Desert (July 4th dawn) |
| Sunset | Santorini (August), Amazon Basin (March) |
| Blue Hour | Paris (January pre-dawn), Chicago (October post-sunset) |
| Winter / Low Angle | Winter Solstice London, Arctic Noon Reykjavik |
| Summer / High Angle | Summer Solstice Helsinki (near-midnight sun) |
| Other | Afternoon Sydney (Southern Hemisphere January) |

---

## How It Works

### Solar Position Algorithm
The calculation uses the NOAA Solar Calculator algorithm, the same method used in the official NOAA spreadsheet tools. It converts your local time to Universal Time, computes the Julian Day, then applies the standard astronomical corrections for geometric mean longitude, equation of centre, apparent longitude, obliquity, right ascension, declination, and the equation of time. A final atmospheric refraction correction is applied to the elevation angle.

The result is an azimuth (degrees clockwise from true North) and an elevation (degrees above the horizon, negative if the sun is below).

### Driver Node Injection
PSA stores its sun direction in linked sockets on its group node. Directly setting `default_value` on a linked socket has no effect in Blender — the link overrides it. PSA Sun Control solves this by injecting small **driver nodes** (`NC_PSA_`) into the World node tree and linking them to PSA's sun sockets. These driver nodes hold the computed values and are permanently connected until you click **Restore PSA Controls**.

Two PSA socket interfaces are supported:
- **Current PSA** — `SunVec` (VECTOR) + `SunCosTheta` (VALUE). A CombineXYZ node drives the direction vector; a Value node drives the cosine.
- **Older PSA** — `Sun Elevation` (VALUE) + `Sun Rotation` (VALUE). Two separate Value nodes are used.

The three-stage node finder handles both automatically — it checks your manually selected node first, then falls back to known PSA name fragments, then to structural socket detection.

### Restore PSA Controls
Clicking **Restore PSA Controls** removes all `NC_PSA_` driver nodes from the World tree and disconnects their links, returning PSA's sockets to their original state for manual control.

---

## Troubleshooting

**Panel not visible**
Make sure you are in **World Properties** (the globe icon in the Properties panel sidebar), not Object or Scene properties.

**"PSA Node Not Found" after scanning**
- Confirm PSA is installed and enabled
- Make sure a World is set in your scene (World Properties → World dropdown)
- Make sure **Use Nodes** is enabled on the World
- Click **Scan World Nodes** — check the System Console (Window → Toggle System Console) for a full socket listing
- If your PSA node was detected but sun sockets didn't match, select it manually from the dropdown and click Calculate & Apply — the console output will show what socket names PSA is using

**Sun position looks wrong**
- Double-check that you selected the correct timezone — DST variants (EDT, CDT, PDT) are separate entries from their standard counterparts (EST, CST, PST)
- Confirm your latitude and longitude signs — North and East are positive, South and West are negative
- Verify the date is correct, especially the year

**PSA controls are greyed out / unresponsive after using the addon**
Click **Restore PSA Controls** to remove the driver nodes. PSA's sliders will become active again.

**Live Update causing slow viewport**
Disable Live Update and use the Calculate & Apply button manually instead.

---

## Timezone Reference

| Identifier | Offset | Zone |
|---|---|---|
| UTC+0 | 0 | UTC / GMT |
| EST | −5 | Eastern Standard |
| EDT | −4 | Eastern Daylight |
| CST | −6 | Central Standard |
| CDT | −5 | Central Daylight |
| MST | −7 | Mountain Standard |
| MDT | −6 | Mountain Daylight |
| PST | −8 | Pacific Standard |
| PDT | −7 | Pacific Daylight |
| AKST | −9 | Alaska Standard |
| HST | −10 | Hawaii Standard |
| BRT | −3 | Brasilia |
| WET | 0 | Western European |
| CET | +1 | Central European |
| CEST | +2 | Central European Summer |
| EET | +2 | Eastern European |
| EEST | +3 | Eastern European Summer |
| MSK | +3 | Moscow |
| IST_IN | +5.5 | India Standard |
| CST_CN | +8 | China Standard |
| JST | +9 | Japan Standard |
| KST | +9 | Korea Standard |
| AEST | +10 | Australian Eastern Standard |
| AEDT | +11 | Australian Eastern Daylight |
| NZST | +12 | New Zealand Standard |

---

*PSA Sun Control Extension v1.0 — © AGW Entertainment — Compatible with Blender 5.1+*
