# SmartHomeShop LD2450 packages

Shared ESPHome configuration packages for all SmartHomeShop products with an
HLK-LD2450 mmWave radar (UltimateSensor V1/V2, UltimateSensor Mini V1 and
CeilSense).

Built on the [native ESPHome ld2450 component](https://esphome.io/components/sensor/ld2450.html),
so no external component is needed. Products include these packages instead of
duplicating the radar configuration per product.

## Packages

### `packages/ld2450-base.yaml`

The radar itself: UART, the ld2450 component and the standard entity set —
presence/motion/still binary sensors, target counts and per-target X/Y/speed/
distance sensors (used by the SmartHomeShop panel for live tracking).

UART pins are configurable per product:

```yaml
substitutions:
  ld2450_uart_tx_pin: GPIO13
  ld2450_uart_rx_pin: GPIO14

packages:
  ld2450_base: github://smarthomeshop/ld2450/packages/ld2450-base.yaml@main
```

### `packages/ld2450-polygon-zones.yaml`

The SmartHomeShop zones layer on top of the base package:

- **4 polygon detection zones** with presence + target count entities
- **2 polygon exclusion zones** (targets inside are ignored)
- **2 entry lines** with direction-aware crossing detection
- **People counter** (persistent across reboots) with reset button/action
- **Last crossing direction** text sensor (`in` / `out`)
- `Polygon Zones Enabled` switch to toggle the whole layer

Zone definitions are stored in text entities and survive reboots. The engine
tracks all three LD2450 targets and is shared by every SmartHomeShop product
that uses this radar. Definitions are pushed from the SmartHomeShop panel in
Home Assistant via these API actions:

| Action | Variables | Format |
|---|---|---|
| `set_polygon_zone` | `zone_id` (1-4), `polygon` | `"x1:y1;x2:y2;x3:y3;..."` (mm, max 20 points) |
| `set_polygon_exclusion` | `zone_id` (1-2), `polygon` | same as above |
| `set_entry_line` | `line_id` (1-2), `line_data` | `"x1:y1;x2:y2;left"` or `"...;right"` |
| `reset_people_count` | - | - |

```yaml
packages:
  ld2450_base: github://smarthomeshop/ld2450/packages/ld2450-base.yaml@main
  ld2450_zones: github://smarthomeshop/ld2450/packages/ld2450-polygon-zones.yaml@main
```

The zones package requires the base package. Its target count, coordinate scale
and target sensor IDs are substitutions, so product repositories do not need
their own copy of the zone engine.

### `packages/ld2450-ceilsense.yaml`

The CeilSense product profile uses ESPHome's native LD2450 component while
preserving all original CeilSense entity IDs. This is important because its
cloud package, diagnostics and startup self-test use those IDs.

CeilSense combines this profile with `ld2450-polygon-zones.yaml`. The radar
still publishes raw floor-plane coordinates. The SmartHomeShop Room Designer
applies the ceiling height, downward mounting orientation and field-of-view
projection when it draws the room and zones.

Ceiling mounting is deliberately not defined in this radar repository. Products
combine their radar profile with the universal
[`smarthomeshop/radar-mounting`](https://github.com/smarthomeshop/radar-mounting)
packages. This keeps mounting, projection and geometry defaults identical for
LD2450, LD2460 and LD6002B products.

```yaml
packages:
  radar: github://smarthomeshop/ld2450/packages/ld2450-ceilsense.yaml@main
  zones: github://smarthomeshop/ld2450/packages/ld2450-polygon-zones.yaml@main
  mounting: github://smarthomeshop/radar-mounting/packages/mounting/ceiling.yaml@main
  radar_profile: github://smarthomeshop/radar-mounting/packages/radars/ld2450.yaml@main
```

## Example

See [`examples/ultimatesensor-mini-v1.yaml`](examples/ultimatesensor-mini-v1.yaml)
for a complete device configuration.

## Products

| Product | Radar | Targets | UART TX | UART RX | Polygon zones |
|---|---|---:|---|---|---|
| UltimateSensor V2 | LD2450 | 3 | GPIO11 | GPIO12 | Yes |
| UltimateSensor Mini V1 | LD2450 | 3 | GPIO13 | GPIO14 | Yes |
| UltimateSensor Mini V2 | LD2450 | 3 | GPIO18 | GPIO19 | Yes |
| CeilSense | LD2450, ceiling mounted | 3 | GPIO13 | GPIO14 | Yes |
