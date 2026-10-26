# mcp-tirepressure

[![CI](https://github.com/StefanOOEOrg/mcp-tirepressure/actions/workflows/ci.yml/badge.svg)](https://github.com/StefanOOEOrg/mcp-tirepressure/actions/workflows/ci.yml)
[![M8ven Score](https://m8ven.ai/badge/mcp/stefanooeorg-mcp-tirepressure-1jh6p4?v=8a5171e131c693844949f6649aa28a75)](https://m8ven.ai/mcp/stefanooeorg-mcp-tirepressure-1jh6p4?s=readme)

Local MCP server (FastMCP, stdio) implementing a SRAM-like tire pressure formula
as tools for the Hermes Agent.

> **Note:** This implementation is based on the formula from the
> [SRAM AXS Tire Pressure Calculator](https://axs.sram.com/tire-pressure).
> The formula was reverse-engineered from the SRAM web client (August 2026).
> **Not an official SRAM product.**

## Safety Notice

> **Important:** The calculated tire pressures are recommendations based on a
> simplified physical formula. They do **not** replace the manufacturer's
> specifications on the tire or your own riding feel. Always check the
> **maximum tire pressure** (on the tire sidewall) and, when in doubt,
> increase the pressure by 0.2–0.3 bar. **Use at your own risk.**

## Setup

```bash
python -m venv .venv
.venv/bin/pip install -e ".[dev]"
```

## Tools

- `get_bikes()` — all 5 bike profiles
- `get_bike(name)` — a single bike profile
- `calc_pressure(bike, rider_weight, bike_weight, surface)` — front/rear pressure in bar
- `get_recommended_pressure(bike, rider_weight, bike_weight, lat, lon)` — with live weather

## Registration in Hermes

```yaml
mcp_servers:
  mcp-tirepressure:
    command: /path/to/mcp-tirepressure/.venv/bin/python
    args: ["-m", "mcp_tirepressure"]
```

## Tool Reference

### `get_bikes()`
Lists all 5 configured bike profiles.
Return: `[{name, tire_width, inner_rim_width, wheel_diameter, ride_style, rim_type, tire_casing, bike_weight}, ...]`

### `get_bike(name: str)`
Returns a single bike profile.
Args: `name` — key from `get_bikes()` (e.g. `"bmc-teamachine"`).
Return: profile as dict.

### `calc_pressure(bike, rider_weight, bike_weight, surface="DRY")`
Calculates the recommended tire pressure (front/rear) in bar.
Args:
- `bike` — bike name
- `rider_weight` — rider weight in kg
- `bike_weight` — bike weight in kg
- `surface` — `"DRY"`, `"WET"`, or `"SNOW"` (default: `"DRY"`)
Return: `{bike, surface, front_bar, rear_bar, notes}`

### `get_recommended_pressure(bike, rider_weight, bike_weight, lat, lon)`
One-call recommendation with live weather (Open-Meteo).
Args:
- `bike`, `rider_weight`, `bike_weight` — as above
- `lat`, `lon` — coordinates (e.g. Vienna: 48.2082, 16.3738)
Return: `{bike, surface, weather_reason, front_bar, rear_bar, notes}`

## Formula

The calculation is based on the formula from the
[SRAM AXS Tire Pressure Calculator](https://axs.sram.com/tire-pressure):

```
P = 10^8.6847 / C^1.3046 × (1 + (2.2 × (W_bike + W_rider) - 180) × 0.0025)
    × R_pos × R_rim × R_style × R_surface × R_casing × 68.9476 × 0.001
```

- `C` — tire circumference in mm (derived from wheel diameter + effective width)
- `R_pos` — front/rear wheel factor (0.94 / 1.0)
- `R_rim` — rim type factor (CROCHET, STRAIGHT_SIDE, TUBES, TUBULAR)
- `R_style` — riding style factor (ROAD=1.0, GRAVEL=0.9, MTB=0.9–1.1)
- `R_surface` — surface condition (DRY=1.0, WET=0.9, SNOW=0.5)
- `R_casing` — tire casing (THIN=1.025, STANDARD=1.0, REINFORCED=0.95, DOUBLE=0.9)
- `68.9476` — PSI → bar conversion (÷ 1000)

Details: `src/mcp_tirepressure/core.py`

## Hookless Cap

For `rim_type == STRAIGHT_SIDE` (hookless), pressure is capped at max
**4.96 bar (72 PSI)**. A note in the `notes` field indicates whether the
cap was triggered.

## Example (CLI)

```bash
.venv/bin/python -c "
from mcp_tirepressure.server import calc_pressure
import json
print(json.dumps(calc_pressure('bmc-teamachine', 93.0, 8.0, 'DRY'), indent=2))
"
```

## License

[MIT](LICENSE)