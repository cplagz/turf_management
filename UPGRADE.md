# Upgrade Guide: v1.0 → v2.0

## Quick Summary

**What's Changed:**
- Lawn areas: `lawn_area_front/back` → `lawn_area_1_size` through `lawn_area_5_size`
- Area names: New `input_text` helpers for custom names
- Input mode: Sliders → Number input fields
- New features: Spray conditions, PGR forecast, lawn type selector
- Configuration: Now uses 3 YAML anchors (was 2)

**Data Migration:**
- Your rates and intervals will need to be manually re-entered
- Your logbook history will remain intact
- Your task dates will be reset (or manually transfer)

---

## Option 1: Clean Install (Recommended)

Best for most users. Takes 15 minutes.

**Steps:**

1. **Document your v1.0 settings**
   - Write down front lawn size
   - Write down back lawn size
   - Screenshot or note all rates
   - Screenshot or note all intervals
   - Note last application dates if needed

2. **Remove v1.0**
   ```
   Delete: config/packages/lawn_care.yaml
   Delete: turf_management dashboard
   ```

3. **Install v2.0**
   - Follow CONFIGURATION.md
   - Configure 3 YAML anchors (temperature, notification, weather)
   - Copy files to packages/ and www/
   - Restart HA
   - Install dashboard

4. **Transfer Settings**
   - Set lawn type: "Warm Season (C4)" (Perth default)
   - Area 1 name: "Front Yard", size: [your v1.0 front size]
   - Area 2 name: "Back Yard", size: [your v1.0 back size]
   - Areas 3-5: Leave at 0m²
   - Re-enter all rates from your notes
   - Re-enter all intervals from your notes
   - Set GDD target (was in v1.0, probably 200-250)

5. **Initialize**
   - Seed temperature tracking (see Setup Guide)
   - Optionally log last application dates manually

**Advantages:**
- Clean start, no conflicts
- Guaranteed to work
- Fresh entity IDs

**Disadvantages:**
- 15 minutes of setup
- Manual data transfer

---

## Option 2: Side-by-Side (Advanced)

Run both versions temporarily to compare.

**Steps:**

1. **Rename v1.0 package**
   ```
   config/packages/lawn_care.yaml → lawn_care_v1.yaml
   ```

2. **Install v2.0**
   ```
   config/packages/lawn_care_v2.yaml
   ```

3. **Update entity IDs in v1.0**
   All v1.0 entities need prefixes to avoid conflicts:
   ```
   lawn_area_front → lawn_v1_area_front
   lawn_area_back → lawn_v1_area_back
   (repeat for all entities...)
   ```

4. **Restart & Compare**
   - Both systems run simultaneously
   - Transfer values manually
   - Remove v1.0 when satisfied

**Advantages:**
- Can compare systems
- Gradual migration

**Disadvantages:**
- Complex setup
- Doubled entity count
- Must manually prevent conflicts

---

## Entity Mapping (v1.0 → v2.0)

### Lawn Areas

| v1.0 | v2.0 |
|------|------|
| `input_number.lawn_area_front` | `input_number.lawn_area_1_size` |
| `input_number.lawn_area_back` | `input_number.lawn_area_2_size` |
| (none) | `input_number.lawn_area_3_size` |
| (none) | `input_number.lawn_area_4_size` |
| (none) | `input_number.lawn_area_5_size` |
| (none) | `input_text.lawn_area_1_name` ("Front Yard") |
| (none) | `input_text.lawn_area_2_name` ("Back Yard") |

### New in v2.0

| Entity | Purpose |
|--------|---------|
| `input_select.lawn_type` | Grass type selector |
| `sensor.lawn_total_area` | Sum of all areas |
| `sensor.lawn_spray_conditions` | Wind/rain suitability |
| `sensor.lawn_wind_speed_forecast` | 24hr wind |
| `sensor.lawn_rain_forecast` | 24hr rain |
| `sensor.lawn_next_spray_window` | Next good spray day |
| `sensor.lawn_gdd_projection_5day` | 5-day GDD forecast |
| `sensor.lawn_pgr_forecast_date` | Estimated PGR date |
| `sensor.lawn_pgr_days_until` | Days until target |

### Unchanged (Same Entity IDs)

- All rates: `input_number.lawn_rate_*`
- All intervals: `input_number.lawn_interval_*`
- All datetimes: `input_datetime.lawn_last_*`
- All scripts: `script.lawn_log_*`
- GDD sensors: `sensor.lawn_gdd_today`, `input_number.pgr_gdd_accumulated`
- Temperature: `input_number.lawn_temp_daily_max/min`

**Note:** Entity IDs are the same, but the mode changed from `slider` to `box`.

---

## Configuration Changes

### v1.0 Configuration

```yaml
# 2 anchors
package.lawn_temp_sensor: &temp_sensor sensor.weatherflow_sensors_temperature
package.lawn_notify_service: &notify_service notify.mobile_app_marcs_iphone
```

### v2.0 Configuration

```yaml
# 3 anchors (added weather)
package.lawn_temp_sensor: &temp_sensor sensor.weatherflow_sensors_temperature
package.lawn_notify_service: &notify_service notify.mobile_app_marcs_iphone
package.lawn_weather: &weather_forecast weather.home  ← NEW
```

**Action Required:**
- Add 3rd anchor for weather forecast entity
- Used by spray conditions and PGR forecast features

---

## Dashboard Changes

### v1.0 Dashboard
- Single view
- Fixed "Front" and "Back" display
- Slider controls

### v2.0 Dashboard
- Two views: Dashboard + Settings
- Dynamic area display (shows only areas > 0m²)
- Number input controls
- New cards: Lawn Type, Spray Conditions, PGR Forecast
- Reorganized settings into dedicated tab

**Action Required:**
- Delete old dashboard completely
- Install new v2.0 dashboard
- Update temperature sensor entity (line ~95)

---

## Data Retention

### Will Be Kept
✅ Logbook entries (historical record)  
✅ Automation history  
✅ Entity history graphs  

### Will Be Lost (If Clean Install)
❌ Last application dates (unless manually transferred)  
❌ Current GDD accumulation (reset to 0)  
❌ Configured rates (unless noted and re-entered)  
❌ Configured intervals (unless noted and re-entered)  

### Recommendation
Screenshot your v1.0 dashboard before upgrading to capture all current values.

---

## Troubleshooting Upgrade Issues

### Both v1.0 and v2.0 entities showing

**Cause:** Didn't remove v1.0 package  
**Fix:** Delete `config/packages/lawn_care.yaml` (v1.0), keep only `lawn_care.yaml` (v2.0), restart

### v2.0 entities not appearing

**Cause:** YAML error or packages not enabled  
**Fix:** Check YAML (Developer Tools → YAML → Check Configuration), verify packages enabled in configuration.yaml

### Dashboard shows "Entity not found"

**Cause:** Dashboard references v1.0 entity IDs  
**Fix:** Delete old dashboard, install fresh v2.0 dashboard

### Spray conditions show "unavailable"

**Cause:** Weather entity not configured  
**Fix:** Add 3rd anchor to lawn_care.yaml (see Configuration Changes above)

---

## Post-Upgrade Checklist

After upgrading:

- [ ] v1.0 package removed
- [ ] v2.0 package in `config/packages/lawn_care.yaml`
- [ ] SVG background in `config/www/turf_bg.svg`
- [ ] 3 YAML anchors configured
- [ ] HA restarted successfully
- [ ] Old dashboard deleted
- [ ] New v2.0 dashboard installed
- [ ] Temperature sensor updated in dashboard
- [ ] Lawn type selected
- [ ] Areas 1-2 configured with your old front/back values
- [ ] All rates re-entered
- [ ] All intervals re-entered
- [ ] GDD target set
- [ ] Temperature tracking initialized
- [ ] Spray conditions showing data (not "unavailable")
- [ ] PGR forecast showing data (not "unknown")

---

## Need Help?

If upgrade fails:
1. Check Developer Tools → YAML for errors
2. Review `home-assistant.log` for specific errors
3. Verify all 3 anchors configured
4. Confirm weather integration provides forecast data
5. Try clean install instead of side-by-side

For Perth users: Default to "Warm Season (C4)" lawn type.
