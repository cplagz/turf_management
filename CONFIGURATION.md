# Configuration Guide — v2.0

## Quick Configuration (3 Steps)

### Step 1: Edit lawn_care.yaml (Lines 24-26)

```yaml
package.lawn_temp_sensor: &temp_sensor sensor.YOUR_TEMP
package.lawn_notify_service: &notify_service notify.YOUR_NOTIFY
package.lawn_weather: &weather_forecast weather.YOUR_WEATHER
```

**How to find your entities:**

**Temperature Sensor:**
- Settings → Devices & Services → Entities
- Search "temperature"
- Look for outdoor sensor
- Example: `sensor.outdoor_temperature`

**Notification Service:**
- Developer Tools → Services
- Search "notify"
- Look for `notify.mobile_app_*`
- Example: `notify.mobile_app_johns_phone`

**Weather Forecast:**
- Settings → Devices & Services → Integrations
- Find your weather integration
- Typical entity: `weather.home`
- Also works: `weather.forecast_home`, `weather.openweathermap`

### Step 2: Edit lovelace_dashboard.yaml (Line ~95)

Find the temperature graph:
```yaml
- entity: sensor.weatherflow_sensors_temperature  ← Change this
  name: Current
```

Change to your sensor:
```yaml
- entity: sensor.outdoor_temperature  ← Your sensor from Step 1
  name: Current
```

### Step 3: Configure Dashboard Settings

After installation, go to Settings tab and configure:

1. **Lawn Type**
   - Choose from dropdown (affects GDD calculation)
   - Perth = "Warm Season (C4)"

2. **Lawn Areas**
   - Name each area
   - Enter size in m²
   - Set unused to 0m²

3. **Rates & Intervals**
   - Based on product labels
   - See README for Perth recommendations

---

## YAML Anchors Explained

The `&` symbol creates a **reusable value**:
```yaml
package.lawn_temp_sensor: &temp_sensor sensor.outdoor_temperature
```

The `*` symbol **references** it:
```yaml
entity_id: *temp_sensor
```

When HA loads the file, every `*temp_sensor` becomes `sensor.outdoor_temperature`.

**You only edit the `&` line. All `*` references update automatically.**

---

## Weather Integration Requirements

The spray forecast and PGR forecast features require:

**Forecast Data Needed:**
- `wind_speed` - for spray conditions
- `precipitation` - for spray conditions
- `temperature` - for PGR forecast
- `templow` - for PGR forecast

**Compatible Integrations:**
- ✅ Met.no (weather.home)
- ✅ OpenWeatherMap
- ✅ Weather.gov
- ✅ AccuWeather
- ⚠️ Some integrations may not provide all fields

**Test Your Integration:**

Developer Tools → Template:
```jinja
{{ state_attr('weather.home', 'forecast') }}
```

Should return array with objects containing temperature, templow, wind_speed, precipitation.

---

## Example Configurations

### Example 1: Simple Setup
```yaml
# Single outdoor sensor, mobile app, met.no weather
package.lawn_temp_sensor: &temp_sensor sensor.outdoor_temperature
package.lawn_notify_service: &notify_service notify.mobile_app_johns_phone
package.lawn_weather: &weather_forecast weather.home
```

### Example 2: WeatherFlow Station
```yaml
# WeatherFlow station, all devices notification
package.lawn_temp_sensor: &temp_sensor sensor.weatherflow_sensors_temperature
package.lawn_notify_service: &notify_service notify.all_devices
package.lawn_weather: &weather_forecast weather.forecast_home
```

### Example 3: Multiple Weather Sources
```yaml
# Ecobee sensor, specific phone, OpenWeatherMap
package.lawn_temp_sensor: &temp_sensor sensor.ecobee_temperature
package.lawn_notify_service: &notify_service notify.mobile_app_pixel_7
package.lawn_weather: &weather_forecast weather.openweathermap
```

### Example 4: No Mobile App
```yaml
# Generic sensor, UI notifications only
package.lawn_temp_sensor: &temp_sensor sensor.backyard_temperature
package.lawn_notify_service: &notify_service notify.persistent_notification
package.lawn_weather: &weather_forecast weather.home
```

---

## Verification Checklist

After configuration:

- [ ] lawn_care.yaml has 3 anchors configured
- [ ] Dashboard temperature sensor updated
- [ ] YAML validation passes (Developer Tools → YAML → Check)
- [ ] Home Assistant restarted successfully
- [ ] Dashboard loads without errors
- [ ] Lawn type selected
- [ ] At least 1 area configured with size > 0m²
- [ ] `sensor.lawn_total_area` shows correct total
- [ ] `sensor.lawn_spray_conditions` shows data (not "unavailable")
- [ ] `sensor.lawn_pgr_forecast_date` shows data (not "unknown")
- [ ] Temperature tracking initialized (see Setup Guide)

---

## Common Issues

### "Entity not found" errors
**Fix:** Check anchor values match actual entity IDs in Settings → Entities

### Spray conditions show "unavailable"
**Fix:** Verify weather integration provides forecast data (see above)

### PGR forecast shows "unknown"
**Fix:** Weather forecast may not include temperature data - check integration

### Multi-area calculations don't show
**Fix:** Set area sizes > 0m² in Settings tab

---

## Advanced: Custom Weather Entity

If your weather integration uses a non-standard entity name:

**Option 1:** Update the anchor
```yaml
package.lawn_weather: &weather_forecast weather.my_custom_weather
```

**Option 2:** Find & Replace in template sensors

Search for: `'weather.home'`  
Replace with: `'weather.my_custom_weather'`

**Files to update:**
- lawn_care.yaml (template sensor section only, ~lines 150-450)

The anchor (`*weather_forecast`) works in automations but not in Jinja templates.
