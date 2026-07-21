# Air Quality Scene Card

An animated Home Assistant dashboard card for **any air-quality sensor** — IKEA ALPSTUGA,
AirGradient, Awair, PurpleAir, ESPHome builds, or anything else that exposes a PM2.5, CO₂,
PM10, VOC-index or NOx-index `sensor` entity. Link a sensor, and the whole scene — sky, hills,
and the mascot's mood — shifts with the air quality, so a glance tells you the state of the air
before you read a number.

<p align="center">
  <a href="https://my.home-assistant.io/redirect/hacs_repository/?owner=keranm&repository=air-quality-scene-card&category=dashboard">
    <img src="https://my.home-assistant.io/badges/hacs_repository.svg" alt="Open the Air Quality Scene Card repository in HACS on your Home Assistant instance.">
  </a>
</p>

<p align="center">
  <img src="images/card-good.png" alt="Air Quality Scene Card — Good" width="480">
</p>

| | | |
|---|---|---|
| ![Good](images/card-good.png) | ![Moderate](images/card-moderate.png) | ![Unhealthy for Sensitive Groups](images/card-usg.png) |
| **Good** | **Moderate** | **Unhealthy for Sensitive Groups** |
| ![Unhealthy](images/card-unhealthy.png) | ![Very Unhealthy](images/card-very-unhealthy.png) | ![Hazardous](images/card-hazardous.png) |
| **Unhealthy** | **Very Unhealthy** | **Hazardous** |

This is the standalone-card sibling of
[airgradient-public](https://github.com/keranm/airgradient-public), which bundles the same UI
with an integration for public AirGradient map locations. This card has **no integration and
no API** — it reads whatever sensor entities you already have.

## Features

- **Compact view**: an animated scene — drifting clouds, a bobbing mascot whose face and
  colour follow the air-quality category — with the current reading and secondary chips
  (temperature, humidity, and every sibling pollutant — CO₂, PM1/2.5/10, PM0.3 count, VOC,
  NOx) auto-discovered from the same device. The `chips` option chooses between showing all
  of them or a minimal set.
- **Tap to expand** into a detail sheet with a colour-coded **24-hour chart**, **12 h / 24 h
  averages**, metric-specific insight tiles, and a **10-days-by-hour heatmap** for the primary
  reading — then, stacked below, a **24-hour chart for every other reading** the sensor
  exposes, so one scroll shows the whole device's last day. All built from Home Assistant's own
  recorder statistics.
- **Five primary metrics**, selectable per card:

  | `metric` | Categories | Insight tiles |
  |---|---|---|
  | `pm25` (default) | US EPA May-2024 PM2.5 (Good → Hazardous) | WHO annual guideline · cigarettes-equivalent |
  | `co2` | Indoor-ventilation (Fresh → Extreme) | Peak-24 h · time above 1000 ppm |
  | `pm10` | US EPA PM10 AQI, 24 h (Good → Hazardous) | Peak-24 h · time above the good range |
  | `voc` | Sensirion VOC Index (Clean → Very High) | Peak-24 h · time above the baseline |
  | `nox` | Sensirion NOx Index (Baseline → High) | Peak-24 h · time above the baseline |

  The CO₂ mode is made for rooms where CO₂ builds up naturally — an office or streaming room
  with the door shut — where ventilation matters more than particulates. The VOC and NOx
  indices are unitless Sensirion scales (100 and 1 are their respective baselines); because
  they carry no `device_class`, the card recognises them by their entity_id.

## Installation

### HACS (recommended)

The quickest way — click the button, which opens your Home Assistant with this repository
pre-filled in HACS:

[![Open in HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=keranm&repository=air-quality-scene-card&category=dashboard)

Then click **Download** and reload your browser when prompted. HACS registers the dashboard
resource automatically — no restart needed for a dashboard card.

<details>
<summary>Or add it manually as a custom repository</summary>

1. In HACS, open the three-dot menu → **Custom repositories**.
2. Add `https://github.com/keranm/air-quality-scene-card` with category **Dashboard**.
3. Find **Air Quality Scene Card**, click **Download**.
4. Reload your browser when prompted.

</details>

### Manual

Copy `dist/air-quality-scene-card.js` to `config/www/` and add a dashboard resource
(Settings → Dashboards → ⋮ → Resources):

```yaml
url: /local/air-quality-scene-card.js
type: module
```

> 💡 **After installing or updating**, hard-refresh your browser once
> (**Cmd/Ctrl + Shift + R**) so it loads the new JavaScript.

## Usage

The card appears in the dashboard card picker as **“Air Quality Scene Card”** with a full
visual editor — pick the metric, pick the sensor, done. Or by YAML:

```yaml
# PM2.5 (default) — e.g. an IKEA ALPSTUGA
type: custom:air-quality-scene-card
entity: sensor.quinns_room_quinn_air_quality_pm2_5
```

```yaml
# CO₂ as the primary reading — for that door-shut streaming room
type: custom:air-quality-scene-card
metric: co2
entity: sensor.quinns_room_quinn_air_quality_carbon_dioxide
name: Quinn's Room
```

```yaml
# A full AirGradient ONE — VOC index headline, minimal chips
type: custom:air-quality-scene-card
metric: voc
entity: sensor.kitchen_air_gradient_voc_index
name: In Kitchen
chips: minimal
```

Every secondary reading (temperature, humidity, and the sibling pollutants) is auto-discovered
from entities on the **same device**, so `entity` is usually the only option you need. If the
metric is omitted, it is inferred from the entity's `device_class` — or, for the unitless VOC
and NOx indices, from its entity_id.

### Options

| Option | Required | Description |
|---|---|---|
| `entity` | ✅ | The sensor to display (PM2.5, CO₂, PM10, VOC index or NOx index). |
| `metric` | – | `pm25`, `co2`, `pm10`, `voc` or `nox`. Default: inferred from the entity. |
| `name` | – | Override the title (defaults to the device name). |
| `chips` | – | `auto` (default — every discovered reading except the headline), `minimal` (temperature, humidity and one pollutant), or an explicit list of reading keys, e.g. `[temperature, humidity, pm25, voc]`. In the visual editor this is **All readings** / **Minimal** / **Customise** — Customise shows a tick-list of every reading plus a sensor picker for each ticked one. |
| `temperature` | – | Explicit temperature entity (otherwise auto-detected). |
| `humidity` | – | Explicit humidity entity (otherwise auto-detected). |
| `co2` / `pm25` / `pm1` / `pm10` / `pm03` / `voc` / `nox` | – | Explicit entity for any secondary chip (otherwise auto-detected). |

## How the history charts work

The expanded view is built entirely from **Home Assistant's recorder statistics**, so the
sensor needs `state_class: measurement` (true for ALPSTUGA and virtually all modern
integrations). The charts **start empty and fill in over time**: the 24-hour chart after a
few hours, the 10-day heatmap and 30-day tiles over the following days. The compact card is
live immediately.

## Try the different states

Use **Developer Tools → States**: pick your sensor, set its **State**, and click **Set
state** — the card recolours instantly (the integration's next poll restores the real value).

| PM2.5 | CO₂ | PM10 | VOC | NOx | Category (tier) |
|---|---|---|---|---|---|
| `2` | `500` | `20` | `80` | `1` | Good / Fresh / Clean / Baseline |
| `20` | `900` | `100` | `150` | `50` | Moderate / Acceptable / Normal |
| `45` | `1200` | `200` | `250` | `200` | Unhealthy for Sensitive Groups / Getting Stuffy / Elevated |
| `80` | `1800` | `300` | `350` | `400` | Unhealthy / Stuffy — Ventilate / High |
| `200` | `3000` | `400` | `450` | – | Very Unhealthy / Poor / Very High |
| `300` | `6000` | `500` | – | – | Hazardous / Extreme |

## Notes on methodology

- **PM2.5 categories** use the [US EPA May-2024 breakpoints](https://www.airnow.gov/aqi/aqi-basics/).
- **WHO comparison** uses the 2021 annual PM2.5 guideline of **5 µg/m³**.
- **Cigarette equivalent** uses the [Berkeley Earth](https://berkeleyearth.org/air-pollution-and-cigarette-equivalence/)
  rule of thumb: a day breathing **22 µg/m³** of PM2.5 ≈ one cigarette.
- **CO₂ categories** follow common indoor-air guidance: outdoor air is ~420 ppm, ~1000 ppm
  is the long-standing "ventilate" line (Pettenkofer number, echoed by ASHRAE 62.1 guidance),
  drowsiness and complaints are typical by 1500–2000 ppm, and **5000 ppm** is the OSHA
  8-hour workplace exposure limit.
- **PM10 categories** use the [US EPA PM10 AQI breakpoints](https://www.airnow.gov/aqi/aqi-basics/)
  (24-hour average).
- **VOC & NOx indices** are [Sensirion's dimensionless Gas Index](https://sensirion.com/products/catalog/SGP41)
  scales (1–500). The VOC Index self-calibrates so that **100** is the recent running-average
  baseline for the room; the NOx Index sits at **1** in clean air and rises only during
  nitrogen-oxide events (cooking, combustion, traffic). Because they self-reference, treat
  them as *"more/less than usual here"* rather than absolute health thresholds.
