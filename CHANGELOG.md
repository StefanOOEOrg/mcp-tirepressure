# Changelog

## 0.1.0 — 2026-10-24

Initial release.

- `get_bikes()` / `get_bike(name)` — five bike profiles (tire width, inner rim width, wheel
  diameter, ride style, rim type, tire casing, bike weight).
- `calc_pressure(bike, rider_weight, bike_weight, surface="DRY")` — front/rear pressure in bar from
  a SRAM-AXS-like formula, with `surface` dry/wet/snow.
- `get_recommended_pressure(bike, rider_weight, bike_weight, lat, lon)` — the same calculation with
  live weather from Open-Meteo (the weather decides the surface).
- Local, keyless MCP server over stdio (FastMCP); no account, no open port.
- CI: pytest + ruff on Python 3.10 and 3.14.
