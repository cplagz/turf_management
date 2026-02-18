# Turf Management v2.0 — Feature List

## Core Multi-Area System
- 5 configurable lawn areas with custom names
- Areas set to 0m² are automatically hidden
- All calculations loop through active areas only
- New sensor: `sensor.lawn_total_area`

## Generic Configuration (YAML Anchors)
- Temperature sensor: Edit 1 line in lawn_care.yaml
- Notification service: Edit 1 line in lawn_care.yaml  
- Weather forecast: Edit 1 line in lawn_care.yaml
- All references auto-update (21+ locations)

## Enhanced UI
- Number input fields (not sliders) for precise values
- Lawn type selector with 5 options
- Spray conditions card with wind/rain analysis
- PGR forecast card showing projected application date
- 5-day GDD projection display

## Spray Conditions Feature
NEW SENSORS:
- `sensor.lawn_wind_speed_forecast` - Next 24hr max wind
- `sensor.lawn_rain_forecast` - Next 24hr rain total
- `sensor.lawn_spray_conditions` - Good/Marginal/Poor rating
- `sensor.lawn_next_spray_window` - Next suitable spray day

LOGIC:
- Good: Wind <15 km/h, Rain <2mm
- Marginal: Wind 15-25 km/h OR Rain 2-5mm  
- Poor: Wind >25 km/h OR Rain >5mm

## PGR Forecast Feature
NEW SENSORS:
- `sensor.lawn_gdd_projection_5day` - Sum of next 5 days GDD
- `sensor.lawn_pgr_forecast_date` - Estimated date when GDD target reached
- `sensor.lawn_pgr_days_until` - Days until PGR needed

LOGIC:
- Uses weather forecast max/min temps
- Calculates daily GDD for next 5-7 days
- Projects when accumulated GDD will hit target
- Accounts for current accumulation

## Lawn Type Selector
OPTIONS:
- Cool Season (C3) - Base temp 5°C
- Warm Season (C4) - Base temp 10°C  
- Bermuda - Base temp 10°C
- Kikuyu - Base temp 10°C
- Buffalo - Base temp 10°C

IMPACT:
- Adjusts GDD base temperature automatically
- Shows in dashboard for reference
- Can be used for future conditional logic

## Entity Requirements
REQUIRED (configure via anchors):
- Temperature sensor (outdoor, °C)
- Notification service
- Weather forecast integration (weather.home or similar)

OPTIONAL:
- Rain sensor (for real-time spray decisions)
- Wind sensor (overrides forecast if available)

## Migration from v1.0
1. Note your v1.0 area sizes
2. Install v2.0
3. Configure anchors (3 lines)
4. Set Area 1 = old Front
5. Set Area 2 = old Back
6. Set lawn type
7. Areas 3-5 remain at 0m² unless needed
