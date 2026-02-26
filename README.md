# Perth Turf Management System v3.8.0

A comprehensive Home Assistant package for managing warm-season turf in Perth, Western Australia. Tracks applications, schedules reminders, calculates GDD, estimates irrigation, monitors inventory, and provides a full product database of 106 Australian turf products across 10 categories.

---

## Features

### Product Database (106 products)
- **10 categories:** Kelp/Seaweed, Granular Fertiliser, Liquid Fertiliser, Humate/Humic Acid, Soil Wetter, Pre-emergent, Insecticide, Fungicide, Herbicide, and Plant Growth Regulator (PGR)
- Auto-fill application rates, units, and nitrogen fraction when selecting a product
- Per-category application method selector (Foliar, Soil Drench, Granular, Spot Spray, etc.)
- Product info sensors exposing active ingredient, notes, and rate ranges
- Custom product support — override any category with your own name, AI, and notes

### Growing Degree Days (GDD)
- Daily GDD accumulation based on your local temperature sensor
- 5-day GDD projection using weather forecast data
- PGR (Primo Maxx / Astro) reapplication tracking with configurable GDD threshold
- GDD-based push notification when PGR reapplication is due

### Scheduling & Reminders
- Configurable reminder intervals per category (days between applications)
- Per-category notification toggles — silence any reminder independently
- Last-applied date tracking via `input_datetime` entities
- Push notifications via mobile app (configurable notify service)

### Inventory Tracking
- Stock level per product with configurable low-stock threshold
- Automatic stock deduction on application logging
- Low stock alert notifications
- Stock displayed in dashboard units (ml, g, L, kg)

### Irrigation
- ET₀-based irrigation estimate using temperature, humidity, and wind
- Suggested irrigation minutes per zone based on lawn area and sprinkler output
- Soil moisture monitoring integration
- Irrigation reminder notifications

### Nitrogen Tracking
- Per-application nitrogen calculation using product N-fraction and rate
- Seasonal nitrogen accumulator (resets annually)
- Season Review chart: Nitrogen Applied vs Average Temperature (requires `apexcharts-card`)

### Dashboard (4 views)
1. **Dashboard** — Live conditions, GDD tracking, spray window, task cards, 5-day forecast, irrigation
2. **Settings** — Lawn areas, reminder intervals, notification toggles, stock levels, soil moisture
3. **Products** — Product selection, application methods, product details (Nutrition / Soil & Growth / Protection), rates & units, enabled products, custom products
4. **Season Review** — Monthly temperature trends, Nitrogen vs Temperature dual-axis chart

---

## Requirements

### Home Assistant
- **Version:** 2024.8 or later (uses modern `action:` syntax)
- **Installation type:** Any (HAOS, Container, Core, Supervised)

### Required Integrations
- A temperature sensor (default: `sensor.weatherflow_sensors_temperature`)
- A weather forecast entity (default: `weather.weatherflow_forecast_home`)
- Mobile app for push notifications (default: `notify.mobile_app_marcs_iphone`)

### HACS Frontend Cards (recommended)
- [Mushroom Cards](https://github.com/piitaya/lovelace-mushroom) — used for select cards and template cards
- [Mini Graph Card](https://github.com/kalkih/mini-graph-card) — temperature and GDD graphs
- [card-mod](https://github.com/thomasloven/lovelace-card-mod) — dark green theme styling
- [ApexCharts Card](https://github.com/RomRider/apexcharts-card) — Season Review dual-axis chart (optional)

---

## Installation

### 1. Copy the package file

Place `lawn_care.yaml` into your Home Assistant packages directory:

```
config/packages/lawn_care.yaml
```

If you haven't enabled packages yet, add this to your `configuration.yaml`:

```yaml
homeassistant:
  packages:
    lawn_care: !include packages/lawn_care.yaml
```

### 2. Add the background image (optional)

Place `turf_bg.svg` in your `config/www/` directory. The dashboard references `/local/turf_bg.svg`. If you don't have this file, remove or change the `background:` blocks in the dashboard YAML.

### 3. Import the dashboard

Go to **Settings → Dashboards → Add Dashboard** (or edit an existing one). Open the Raw Configuration Editor (three-dot menu → "Edit in YAML") and paste the contents of `lovelace_dashboard.yaml`.

### 4. Configure for your setup

Edit the three user-config values near the top of `lawn_care.yaml`:

| Setting | Default | Description |
|---------|---------|-------------|
| Temperature sensor | `sensor.weatherflow_sensors_temperature` | Your outdoor temperature entity |
| Notify service | `notify.mobile_app_marcs_iphone` | Your mobile app notification target |
| Weather forecast | `weather.weatherflow_forecast_home` | Weather entity for forecasts |

Search for `# ⚙️ USER-CONFIG` comments throughout the file to locate all references that need updating.

### 5. Restart Home Assistant

A full restart is required on first install to create all entities.

---

## Upgrading from v3.7.0 or earlier

v3.8.0 includes critical bug fixes that require a clean entity refresh:

1. Replace `lawn_care.yaml` with the new version
2. Replace the dashboard YAML (Raw Configuration Editor)
3. Go to **Developer Tools → YAML → Reload Template Entities**
4. (Optional) Delete orphaned entities from **Settings → Devices & Services → Entities** — filter for old names like `sensor.kelp_seaweed_db`, `sensor.granular_fertiliser_product_info`, etc.

If Product Details still shows "unknown", restart Home Assistant fully.

---

## File Reference

| File | Description |
|------|-------------|
| `lawn_care.yaml` | HA package — all entities, sensors, automations |
| `lovelace_dashboard.yaml` | Dashboard — 4-view Lovelace configuration |
| `README.md` | This documentation |

---

## Entity Summary

| Type | Count | Purpose |
|------|-------|---------|
| `input_boolean` | 24 | Product & notification toggles |
| `input_select` | 28 | Lawn type, products, methods, units |
| `input_number` | 45 | Areas, rates, intervals, GDD, stock, irrigation |
| `input_text` | 36 | Custom product fields (name, AI, notes) |
| `input_datetime` | 11 | Last-applied timestamps |
| Template sensors | 33 | GDD, weather, spray conditions, product DB & info |
| Automations | 27 | Logging, reminders, autofill, GDD, stock alerts |

---

## Adding a New Product

Edit `lawn_care.yaml` in two places:

1. **`input_select.lawn_product_db_<category>`** — add the product name to `options:` (before "Custom")

2. **The corresponding DB sensor** (e.g. `Lawn DB Kelp`) — add an entry to the `products` attribute dict following this format:

```yaml
'Product Name': {
  'r': {'Method': [low_rate, high_rate]},
  'u': 'ml',          # unit: ml, g, L, kg
  'a': 'Active ingredient description',
  't': 'Application notes and tips.',
  'n': 0.0            # nitrogen fraction (0–1), 0 if N/A
}
```

Rate values are per 100m². Products can have multiple methods (e.g. `'Foliar': [100, 200], 'Soil Drench': [100, 200]`).

---

## Product Categories & Counts

| Category | Products | Methods |
|----------|----------|---------|
| Kelp / Seaweed | 10 | Foliar, Soil Drench |
| Granular Fertiliser | 31 | Broadcast |
| Liquid Fertiliser | 12 | Foliar, Soil Drench |
| Humate / Humic Acid | 10 | Soil Drench |
| Soil Wetter | 17 | Soil Drench |
| Pre-emergent | 4 | Granular, Foliar Spray, Soil Drench |
| Insecticide | 8 | Granular, Foliar Spray, Soil Drench |
| Fungicide | 4 | Foliar Spray, Curative Drench |
| Herbicide | 8 | Foliar Spray, Spot Spray |
| PGR | 2 | Foliar Spray |

---

## Changelog

### v3.8.0
- Dashboard Products view (View 3) redesigned with 4 logical stacks: Product Selection, Product Details (split into Nutrition / Soil & Growth / Protection), Rates & Units, and Enabled Products & Custom
- Consistent section header styling across all views
- Added category icons to rate/unit entity rows
- Added missing CSS variables to Products view theme block

### v3.7.1
- **Critical fix:** Product Details showing "unknown / None" for all categories. Root cause: sensor names produced wrong entity_ids (e.g. "Kelp / Seaweed DB" → `sensor.kelp_seaweed_db` instead of expected `sensor.lawn_db_kelp`). Renamed all 20 sensors and removed `unique_id` to prevent entity registry conflicts.
- Pre-emergent DB method key corrected: `Liquid Spray` → `Foliar Spray`
- Fungicide DB method key corrected: `Soil Drench` → `Curative Drench`

### v3.7.0
- Jinja2 TemplateSyntaxError fix for apostrophes in product names
- Full product/rate refresh from master CSV (106 products)
- Added 21 new products across Baileys, Plant Doctor, Floratine, Eco Growth, and Ralphy's ranges
- Rate corrections per updated CSV data

### v3.6.0
- Added 16 Billy Goat Lawns / PGF / ICL / Lawn Juice / Sacoa products

### v3.5.0
- Replaced entire product database — 69 Australian turf products
- Insecticide methods updated: Granular, Foliar Spray, Soil Drench

### v3.4.0
- UI-editable custom products — 30 `input_text` entities for name, AI, notes per category
- Conditional custom product cards in Products view

### v3.3.0
- Refactored to single-source-of-truth product database architecture
- Each category's product dict defined once in a `lawn_db_*` sensor

### v3.2.0
- Product Database — auto-fill rates, units, and N-fraction on selection
- Application method selectors
- Product info sensors with AI, notes, and rate ranges

### v3.1.0
- Product enable/disable toggles
- Per-product unit selectors (ml / g / L / kg)

### v3.0.x
- Inventory tracking, ET₀ irrigation, soil moisture, nitrogen tracking
- Season Review dashboard
- HA 2024.8+ migrations

---

## License

Personal use. Not affiliated with any product manufacturer.
