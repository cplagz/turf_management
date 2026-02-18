# 🌿 Perth Turf Management System
### Home Assistant Package · Perth, WA (Sandy Soil / Warm-Season Grass)

---

## 📦 What's Included

| File | Purpose |
|------|---------|
| `lawn_care.yaml` | Main HA package — all helpers, sensors, scripts & automations |
| `lovelace_dashboard.yaml` | Lovelace dashboard YAML to paste into the UI editor |
| `README.md` | This file |

---

## 🚀 Step 1 — Prerequisites

### Install Custom Cards via HACS
You need two custom Lovelace cards. Install them through **HACS → Frontend**:

1. **Mushroom Cards**
   - Search: `Mushroom`
   - Repository: `piitaya/lovelace-mushroom`

2. **Mini Graph Card**
   - Search: `Mini Graph Card`
   - Repository: `kalkih/mini-graph-card`

After installing, go to **Settings → Dashboards** and clear your browser cache
(or hard refresh with `Ctrl+Shift+R` / `Cmd+Shift+R`).

---

## 🚀 Step 2 — Enable Packages in configuration.yaml

Open your `configuration.yaml` and add (or confirm) this block:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Then create the packages directory if it doesn't already exist:
```
config/
└── packages/          ← create this folder
    └── lawn_care.yaml ← place the file here
```

---

## 🚀 Step 3 — Deploy the Package File

1. Copy `lawn_care.yaml` into `config/packages/lawn_care.yaml`
2. In Home Assistant, go to **Developer Tools → Check Configuration**
   - Fix any errors before restarting
3. Go to **Settings → System → Restart** and do a full restart

---

## 🚀 Step 4 — Add the Dashboard

1. Go to **Settings → Dashboards**
2. Click **+ Add Dashboard**
3. Give it a name (e.g. `Turf Management`) and choose an icon (`mdi:grass`)
4. Open the new dashboard, click the **pencil icon** (Edit)
5. Click the **three dots (⋮)** → **Raw Configuration Editor**
6. **Delete** the default content and **paste** the entire contents of `lovelace_dashboard.yaml`
7. Click **Save**

---

## 🚀 Step 5 — First-Time Configuration

After restarting HA, go to your new dashboard and set up the following in **Section 3 (Settings)**:

### Lawn Areas
| Helper | Suggested Value |
|--------|----------------|
| Front Lawn Area | Your front lawn m² |
| Back Lawn Area | Your back lawn m² |

### Application Rates (per 100m²)
Set these based on your product labels. Common Perth starting points:

| Product | Typical Rate |
|---------|-------------|
| Granular Fertiliser | 150–200 g/100m² |
| Liquid Fertiliser | 50–150 ml/100m² |
| Kelp | 30–50 ml/100m² |
| Humate | 30–50 ml/100m² |
| Soil Wetter | 20–50 ml/100m² |
| Pre-emergent | 100–200 ml/100m² |
| Insecticide | 30–50 ml/100m² |
| Fungicide | 30–50 ml/100m² |
| Herbicide | 50–100 ml/100m² |
| PGR | 10–25 ml/100m² |

### Task Intervals
Suggested starting intervals for Perth warm-season grass:

| Task | Suggested Interval |
|------|-------------------|
| Mowing | 7 days (summer) |
| Kelp | 14 days |
| Liquid Fert | 14 days |
| Granular Fert | 60 days |
| Humate | 30 days |
| Soil Wetter | 90 days ⚠️ critical in Perth! |
| Pre-emergent | 120 days |
| Insecticide | 90 days |
| Fungicide | 30 days |
| Herbicide | 90 days |

### PGR / GDD Settings
| Helper | Suggested Value |
|--------|----------------|
| Target GDD | 200–250 GDD (warm-season grass) |
| Accumulated GDD | Leave at 0 to start fresh |

---

## 🌡️ How the GDD Engine Works

**Growing Degree Days (GDD)** measure accumulated heat that drives grass growth.

**Formula used:**
```
GDD = max(((Max Temp + Min Temp) / 2) - 10, 0)
Base temperature: 10°C
```

**What happens automatically:**
- Every night at **23:59**, Today's GDD is calculated from your WeatherFlow sensor
- At **midnight**, that value is added to `pgr_gdd_accumulated`
- When accumulated GDD reaches your **Target GDD**, you get a push notification
  with exact mixing amounts for both lawns
- When you log a **PGR application**, the accumulator **resets to 0** automatically

**Note on Max/Min Temperatures:**
The template sensors attempt to read `max_value` / `min_value` attributes from
your WeatherFlow sensor. If your sensor doesn't expose those attributes, the
templates fall back to the current reading. For accurate GDD, consider using
the `utility_meter` integration or a statistics sensor to properly track daily
max/min. See the optional enhancement section below.

---

## 📱 Notification Setup

All notifications go to `notify.mobile_app_marcs_iphone`.

**To change the notification target:**
Search for `notify.mobile_app_marcs_iphone` in `lawn_care.yaml` and replace it
with your own device's notify service. You can find your device's service name at:
**Developer Tools → Services → search "notify"**

Every notification includes:
- Days since last application
- Exact ml/g to pour for **Front Lawn**
- Exact ml/g to pour for **Back Lawn**

---

## 🧪 Testing Without Waiting

### Test a reminder notification immediately:
1. Go to **Developer Tools → Services**
2. Call `input_datetime.set_datetime` on any `input_datetime.lawn_last_*` entity
3. Set the date far in the past (e.g. 2024-01-01)
4. Trigger the automation manually: **Automations → find the reminder → Run**

### Test the GDD accumulator:
```yaml
# Developer Tools → Services
service: input_number.set_value
target:
  entity_id: input_number.pgr_gdd_accumulated
data:
  value: 240   # Just below your target of 250
```

### Test the GDD accumulation automation:
**Automations → "Lawn: Accumulate Daily GDD" → Run**

---

## 🔧 Optional Enhancements

### Better Max/Min Temperature Tracking
Add these to your `configuration.yaml` for accurate daily max/min from WeatherFlow:

```yaml
sensor:
  - platform: statistics
    name: "WeatherFlow Daily Max Temp"
    entity_id: sensor.weatherflow_sensors_temperature
    state_characteristic: value_max
    sampling_size: 1000
    max_age:
      hours: 24

  - platform: statistics
    name: "WeatherFlow Daily Min Temp"
    entity_id: sensor.weatherflow_sensors_temperature
    state_characteristic: value_min
    sampling_size: 1000
    max_age:
      hours: 24
```

Then update the template sensors in `lawn_care.yaml` to reference:
- `sensor.weatherflow_daily_max_temp`
- `sensor.weatherflow_daily_min_temp`

### Rain Gate (hold notifications if rain forecast)
Add this condition to any reminder automation to skip if rain is coming:

```yaml
condition:
  - condition: numeric_state
    entity_id: sensor.your_rain_forecast_sensor
    below: 2   # Skip if >2mm forecast
```

### Seasonal GDD Targets
Create an automation that lowers `pgr_gdd_target` in autumn/winter:

```yaml
- alias: "Lawn: Set Winter PGR Target"
  trigger:
    - platform: time
      at: "00:00:00"
  condition:
    - condition: template
      value_template: "{{ now().month in [5,6,7,8] }}"  # May–Aug
  action:
    - service: input_number.set_value
      target:
        entity_id: input_number.pgr_gdd_target
      data:
        value: 350  # More GDD needed in cooler months
```

---

## 🗂️ Entity Reference

### Sensors
| Entity | Description |
|--------|-------------|
| `sensor.lawn_daily_max_temp` | Today's max temperature |
| `sensor.lawn_daily_min_temp` | Today's min temperature |
| `sensor.lawn_gdd_today` | Today's calculated GDD |

### Key Helpers
| Entity | Description |
|--------|-------------|
| `input_number.pgr_gdd_accumulated` | Running GDD total since last PGR |
| `input_number.pgr_gdd_target` | GDD threshold to trigger PGR notification |
| `input_text.lawn_task_notes` | Notes field — cleared after each log |

### Scripts (call these to log tasks)
| Script | Description |
|--------|-------------|
| `script.lawn_log_mowing` | Log mow |
| `script.lawn_log_kelp` | Log kelp |
| `script.lawn_log_granular_fert` | Log granular fert |
| `script.lawn_log_liquid_fert` | Log liquid fert |
| `script.lawn_log_humate` | Log humate |
| `script.lawn_log_soil_wetter` | Log soil wetter |
| `script.lawn_log_preemergent` | Log pre-emergent |
| `script.lawn_log_insecticide` | Log insecticide |
| `script.lawn_log_fungicide` | Log fungicide |
| `script.lawn_log_herbicide` | Log herbicide |
| `script.lawn_log_pgr` | Log PGR + reset GDD to 0 |

---

## 🆘 Troubleshooting

**"Entity not found" errors after restart**
→ Run **Developer Tools → Check Configuration** before restarting.
→ Ensure the `packages:` line is correctly set in `configuration.yaml`.

**Cards show "Custom element doesn't exist"**
→ Mushroom Cards or Mini Graph Card not installed via HACS, or browser cache
  needs clearing. Hard-refresh with `Ctrl+Shift+R`.

**GDD not accumulating**
→ Check `sensor.lawn_gdd_today` has a valid numeric state in Developer Tools.
→ Ensure `sensor.weatherflow_sensors_temperature` is available and returning data.

**Notifications not arriving**
→ Confirm your notify service name in Developer Tools → Services.
→ Replace `notify.mobile_app_marcs_iphone` throughout `lawn_care.yaml`.

---

*Perth Turf Management System — built for Home Assistant 2024.1+*
*Sandy soil. Warm-season grass. No shortcuts. 🌿*
