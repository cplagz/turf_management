# Perth Turf Management v2.1 — Complete System

Professional lawn care tracking with GDD calculation, spray forecasting, multi-area support, and automated task reminders.

---
# NEW IN v2.1:
# ✓ Notification Toggle Switches (enable/disable specific alerts)
# ✓ Corrected agronomic application rate ranges and units (kg vs ml)
# ✓ Multi-area support (up to 5 configurable areas)
# ✓ Lawn type selector (affects GDD calculation)
# ✓ Spray conditions forecast (wind + rain analysis)
# ✓ Generic entity configuration (3 YAML anchors)
# ✓ 2024.4+ Weather Forecast Compatibility built-in
---

## 🆕 What's New in v2.0

### Multi-Area Lawn Management
- Configure up to 5 lawn areas with custom names
- Set unused areas to 0m² to hide them automatically
- All calculations show per-area mixing amounts
- New sensor: `sensor.lawn_total_area`

### Spray Conditions Forecast
- Real-time wind speed analysis (next 24hr)
- Rain forecast integration (next 24hr)
- Automated suitability rating: Good/Marginal/Poor
- Next suitable spray window detection (7-day forecast)
- Shows in dashboard and included in spray task notifications

### PGR Application Forecasting
- 5-day GDD projection from weather forecast
- Estimated PGR application date
- Days remaining until target GDD reached
- Updates automatically based on temperature forecast

### Lawn Type Selector
- 5 grass types: Cool Season (C3), Warm Season (C4), Bermuda, Kikuyu, Buffalo
- Automatically adjusts GDD base temperature
- Cool Season = 5°C base, all others = 10°C base

### Enhanced UI/UX
- Number input fields (not sliders) for precise values
- Dedicated settings view with organized configuration
- Spray conditions card showing wind/rain at a glance
- PGR forecast card with estimated application date
- Lawn type selector prominently displayed

### Generic Configuration
- 3 YAML anchors: temperature sensor, notification service, weather forecast
- Edit 3 lines in `lawn_care.yaml` to configure
- Auto-populates 30+ entity references
- Distribution-ready for any Home Assistant setup

---

## Core Features (From v1.0)

✅ GDD tracking engine (temperature-based)  
✅ 11 maintenance task logging with mixing calculations  
✅ Automated task interval reminders  
✅ Mobile app notifications  
✅ Dark botanical theme  
✅ Complete logbook integration  

---

## Quick Start

### Prerequisites

- **Home Assistant** 2024.1+
- **Temperature sensor** (outdoor, °C)
- **Weather integration** (for spray/PGR forecasts)
- **Mobile app** (optional for notifications)
- **HACS custom cards:**
  - Mushroom Cards
  - Mini Graph Card
  - card-mod

### Installation (10 minutes)

**1. Configure the Package**

Edit `lawn_care.yaml` lines 24-26:

```yaml
package.lawn_temp_sensor: &temp_sensor sensor.YOUR_TEMP_SENSOR
package.lawn_notify_service: &notify_service notify.YOUR_NOTIFY
package.lawn_weather: &weather_forecast weather.YOUR_WEATHER
```

Examples:
```yaml
package.lawn_temp_sensor: &temp_sensor sensor.outdoor_temperature
package.lawn_notify_service: &notify_service notify.mobile_app_johns_phone
package.lawn_weather: &weather_forecast weather.home
```

**2. Install Package**

Enable packages in `configuration.yaml`:
```yaml
homeassistant:
  packages: !include_dir_named packages
```

Copy `lawn_care.yaml` to `config/packages/lawn_care.yaml`

**3. Install Background**

Copy `turf_bg.svg` to `config/www/turf_bg.svg`

**4. Restart Home Assistant**

**5. Install Dashboard**

- Settings → Dashboards → Add Dashboard
- Edit → Raw Config Editor
- Paste entire `lovelace_dashboard.yaml`
- Update line ~95 with your temperature sensor
- Save

**6. Configure Your Lawn**

Go to dashboard Settings tab:
- Set lawn type (affects GDD base temperature)
- Configure area names and sizes
- Set application rates per 100m²
- Set task intervals in days
- Set GDD target for PGR

**7. Initialize Temperature Tracking**

Developer Tools → Services:
```yaml
service: input_number.set_value
target:
  entity_id: input_number.lawn_temp_daily_max
data:
  value: 32  # Today's actual high
```

```yaml
service: input_number.set_value
target:
  entity_id: input_number.lawn_temp_daily_min
data:
  value: 18  # Today's actual low
```

---

## New Sensors (v2.0)

### Spray Conditions
- `sensor.lawn_wind_speed_forecast` - Next 24hr max wind (km/h)
- `sensor.lawn_rain_forecast` - Next 24hr rain total (mm)
- `sensor.lawn_spray_conditions` - Good/Marginal/Poor rating
- `sensor.lawn_next_spray_window` - Next suitable spray date

**Logic:**
- **Good:** Wind <15 km/h AND Rain <2mm
- **Marginal:** Wind 15-25 km/h OR Rain 2-5mm
- **Poor:** Wind >25 km/h OR Rain >5mm

### PGR Forecast
- `sensor.lawn_gdd_projection_5day` - Projected GDD next 5 days
- `sensor.lawn_pgr_forecast_date` - Estimated application date
- `sensor.lawn_pgr_days_until` - Days until target reached

### Multi-Area
- `sensor.lawn_total_area` - Sum of all active areas (m²)

### Lawn Type
- `input_select.lawn_type` - Grass type selector (affects GDD)

---

## Configuration Guide

### Lawn Areas

Configure up to 5 areas in Settings tab:

**Area 1-5 Name:**
- Examples: "Front Yard", "Back Yard", "Side Strip", "Pool Area", "North Section"

**Area 1-5 Size:**
- Enter size in m²
- Set to 0 to hide unused areas
- Total shown in dashboard

**Example Setup:**
```
Area 1: "Front Lawn" = 120m²
Area 2: "Back Lawn" = 85m²
Area 3: "Side Yard" = 25m²
Area 4: 0m² (disabled)
Area 5: 0m² (disabled)
Total: 230m²
```

### Lawn Type Selection

**Cool Season (C3)** - Base temp 5°C
- Examples: Ryegrass, Fescue, Bluegrass
- Used in cooler climates

**Warm Season (C4)** - Base temp 10°C  
- Examples: Couch, Kikuyu, Buffalo, Bermuda
- Used in Perth and warm climates

**Bermuda** - Base temp 10°C
- Specific variety of warm-season grass

**Kikuyu** - Base temp 10°C
- Perth favorite, aggressive grower

**Buffalo** - Base temp 10°C
- Shade-tolerant warm-season grass

### Application Rates (per 100m²)

**Perth Sandy Soil Recommendations:**

| Product | Rate | Notes |
|---------|------|-------|
| Granular Fert | 150-200g | Slow-release NPK |
| Liquid Fert | 50-150ml | Bi-weekly |
| Kelp | 30-50ml | Biostimulant |
| Humate | 30-50ml | Soil conditioner |
| Soil Wetter | 20-50ml | **Critical for Perth** |
| Pre-emergent | 100-200ml | Seasonal |
| Insecticide | 30-50ml | As needed |
| Fungicide | 30-50ml | Disease control |
| Herbicide | 50-100ml | Spot treatment |
| PGR | 10-25ml | Growth regulator |

**Always check your product labels.**

### Task Intervals

**Perth Climate Guide:**

| Task | Summer | Winter |
|------|--------|--------|
| Mowing | 5-7 days | 10-14 days |
| Kelp | 14 days | 21 days |
| Liquid Fert | 14 days | 21-30 days |
| Granular Fert | 60 days | 90 days |
| Humate | 30 days | 60 days |
| Soil Wetter | 90 days | 120 days |
| Pre-emergent | 120 days | Seasonal |
| PGR | Use GDD | Use GDD |

### GDD / PGR Settings

**Target GDD:** 200-250 for most warm-season grasses
- Check product label for specific recommendations
- Lower values = more frequent applications
- Higher values = longer intervals between applications

**Base Temperature:** Set automatically by lawn type
- Cool Season = 5°C
- Warm Season = 10°C

---

## Entity Reference

### Input Selects
- `input_select.lawn_type` - Lawn type (affects GDD calculation)

### Input Text (Area Names)
- `input_text.lawn_area_1_name` through `input_text.lawn_area_5_name`
- `input_text.lawn_task_notes` - Notes field for logging

### Input Numbers (29 total)
**Area Sizes:**
- `input_number.lawn_area_1_size` through `input_number.lawn_area_5_size`

**Rates (10):**
- `input_number.lawn_rate_*` (granular_fert, liquid_fert, kelp, humate, soil_wetter, preemergent, insecticide, fungicide, herbicide, pgr)

**Intervals (10):**
- `input_number.lawn_interval_*` (same list as rates)

**GDD:**
- `input_number.pgr_gdd_accumulated`
- `input_number.pgr_gdd_target`

**Temperature:**
- `input_number.lawn_temp_daily_max`
- `input_number.lawn_temp_daily_min`

### Template Sensors
- `sensor.lawn_gdd_today` - Today's GDD (lawn type adjusted)
- `sensor.lawn_daily_max_temp` - Today's max
- `sensor.lawn_daily_min_temp` - Today's min
- `sensor.lawn_total_area` - Sum of all areas
- `sensor.lawn_wind_speed_forecast` - 24hr wind
- `sensor.lawn_rain_forecast` - 24hr rain
- `sensor.lawn_spray_conditions` - Suitability rating
- `sensor.lawn_next_spray_window` - Next good spray day
- `sensor.lawn_gdd_projection_5day` - 5-day GDD projection
- `sensor.lawn_pgr_forecast_date` - Estimated PGR date
- `sensor.lawn_pgr_days_until` - Days until PGR needed

### Scripts (11)
All with multi-area calculation support:
- `script.lawn_log_*` (mowing, kelp, pgr, granular_fert, liquid_fert, humate, soil_wetter, preemergent, insecticide, fungicide, herbicide)

### Automations (15)
- Temperature tracking (3)
- GDD accumulation (1)
- PGR threshold alert (1)
- Task reminders (11) - all include spray conditions when relevant

---

## Troubleshooting

### Spray Conditions Show "Unavailable"

**Cause:** Weather forecast entity not configured or integration offline

**Fix:**
1. Verify weather integration is working (check States tab)
2. Edit `lawn_care.yaml` line 26 with correct weather entity
3. Most HA installations use `weather.home`
4. Check template sensors for `weather.home` references (search & replace if needed)

### PGR Forecast Shows "Unknown"

**Cause:** Weather forecast doesn't provide temperature data

**Fix:**
1. Check `state_attr('weather.home', 'forecast')` in Developer Tools → Template
2. Verify forecast contains `temperature` and `templow` keys
3. Some weather integrations need "forecast" mode enabled

### Multi-Area Calculations Not Showing

**Cause:** Area sizes set to 0m²

**Fix:**
1. Go to Settings tab in dashboard
2. Set area sizes to actual measurements
3. Areas with 0m² are automatically hidden

### Number Inputs Show as Sliders

**Cause:** Old browser cache

**Fix:**
1. Hard refresh (Ctrl+Shift+R)
2. All input_numbers have `mode: box` in v2.0

---

## Migration from v1.0

If upgrading from v1.0:

1. **Backup your data**
   - Note current lawn areas
   - Note current rates and intervals
   - Export logbook if desired

2. **Uninstall v1.0**
   - Remove old `lawn_care.yaml`
   - Remove old dashboard

3. **Install v2.0**
   - Follow Quick Start guide above

4. **Transfer Settings**
   - Area 1 = old Front Yard
   - Area 2 = old Back Yard
   - Set lawn type (probably "Warm Season (C4)" for Perth)
   - Restore your rates and intervals

5. **Key Differences**
   - Areas now numbered 1-5 instead of "front"/"back"
   - GDD base temp now automatic based on lawn type
   - Spray conditions require weather forecast entity
   - Number inputs instead of sliders

---

## Support & Contributions

Created for Perth, Western Australia turf enthusiasts.

**Tested on:** Home Assistant 2024.1+  
**License:** MIT  
**Version:** 2.0 (2026-02-18)

---

## Changelog

### v2.0 (2026-02-18)
✨ Multi-area support (up to 5 configurable areas)  
✨ Spray conditions forecast with wind/rain analysis  
✨ PGR forecast based on 5-day GDD projection  
✨ Lawn type selector (affects GDD base temperature)  
✨ Number input fields (replaced sliders)  
✨ Weather forecast integration  
✨ Dedicated settings dashboard view  
✨ Generic entity configuration (3 YAML anchors)

### v1.0 (2026-02-17)
🚀 Initial release  
🌡️ GDD tracking  
📋 11-task management  
🎨 Botanical theme  
🧮 Automatic mixing calculations
