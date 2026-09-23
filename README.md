# survey-cogo

A pure-Python, dependency-free **coordinate geometry (COGO) engine** for surveying: inverses, forwards, intersections, traverse computation and adjustment, and polygon area.

This is module one of a larger surveying + remote sensing suite. It is engine-only on purpose — no UI, no file I/O — so it can be imported by scripts, embedded in a desktop app, or installed as a library.

## Features

- **Angles** — DMS ↔ decimal degrees, DMS string parsing (`122°30'15"`), azimuth ↔ quadrant bearing conversion
- **Inverse** — azimuth and horizontal distance between two known points
- **Forward** — point by azimuth and distance from a known point
- **Intersections** — bearing-bearing (two rays) and distance-distance (two circles, both solutions)
- **Traverses** — turned angles → azimuths, open/closed traverse runs, angular and linear misclosure, relative precision (`1:X`), Compass (Bowditch) adjustment
- **Area** — polygon area by the coordinate method, perimeter

## Installation

Requires Python 3.9+.

```bash
# From source
pip install .

# Editable install (for development)
pip install -e .
```

## Quickstart

```python
from cogo import Point, Leg, inverse, forward, run_traverse, compass_adjust, polygon_area

# Inverse: azimuth and distance between two control points
a = Point(northing=1000.0, easting=1000.0, name="A")
b = Point(northing=1100.0, easting=1200.0, name="B")
inv = inverse(a, b)
print(f"Azimuth {inv.azimuth:.4f}°, distance {inv.distance:.3f}")
# Azimuth 63.4349°, distance 223.607

# Forward: stake a point 250 units from A on azimuth 63.4349°
p = forward(a, 63.4349, 250.0, name="P1")

# Traverse: run a closed loop, then Bowditch-adjust it
legs = [Leg(0, 100), Leg(90, 100), Leg(180, 100), Leg(270, 100)]
pts = run_traverse(a, legs)
adjusted = compass_adjust(a, legs)   # closes exactly on A

# Area of a parcel
parcel = [Point(0, 0), Point(0, 100), Point(100, 100), Point(100, 0)]
print(polygon_area(parcel))  # 10000.0
```

Working from field angles instead of azimuths:

```python
from cogo import azimuths_from_turned_angles, angular_misclosure

# Angles turned to the right, starting on azimuth 90°
azimuths = azimuths_from_turned_angles(90, [90, 90, 90, 90], turn="right")
legs = [Leg(az, 100) for az in azimuths[1:]]

# Check a closed 4-sided loop
print(angular_misclosure([90, 90, 90, 90], n_sides=4))  # 0.0
```

## API reference

### `cogo.angles`

| Function | Description |
|---|---|
| `dms_to_decimal(d, m=0, s=0)` | DMS components → decimal degrees |
| `parse_dms("122°30'15\"")` | Parse a DMS string → decimal degrees |
| `decimal_to_dms(dec)` | Decimal degrees → `(degrees, minutes, seconds)` |
| `format_dms(dec)` | Decimal degrees → `"122°30'15.0\""` |
| `normalize_angle(a)` | Normalize to `[0, 360)` |
| `azimuth_to_bearing(az)` | `135` → `"S 45°00'00.0\" E"` |
| `bearing_to_azimuth("S 45°00'00\" E")` | → `135.0` |

### `cogo` core

| Function | Description |
|---|---|
| `inverse(p1, p2)` | → `Inverse(azimuth, distance)`; raises on coincident points |
| `forward(p, azimuth, dist, name="")` | Point at distance along azimuth |
| `bearing_bearing(p1, az1, p2, az2)` | Intersect two rays; raises if parallel |
| `distance_distance(p1, r1, p2, r2)` | Intersect two circles → both points; raises if no intersection |

### `cogo.traverse`

| Function | Description |
|---|---|
| `azimuths_from_turned_angles(start_az, angles, turn="right")` | Field angles → forward azimuths |
| `run_traverse(start, legs, names=None)` | → `[start, p1, p2, …]` |
| `angular_misclosure(angles, n_sides, interior=True)` | Observed − theoretical angle sum |
| `linear_misclosure(start, end_computed)` | → `LinearMisclosure(d_northing, d_easting, linear_error)` |
| `relative_precision(error, length)` | → `(ratio, "1:X")` |
| `compass_adjust(start, legs, names=None)` | Bowditch adjustment; closes exactly |

### `cogo.area`

| Function | Description |
|---|---|
| `polygon_area(points)` | Shoelace area (absolute value) |
| `polygon_perimeter(points)` | Closed perimeter length |

### `cogo.Point`

Frozen dataclass: `Point(northing, easting, name="", elevation=0.0)`.

## Conventions

- Coordinates are plane **northing (Y) / easting (E)**, any consistent linear unit.
- Azimuths are **decimal degrees, clockwise from north**, normalized to `[0, 360)`.
- Distances are horizontal, in the same unit as the coordinates.
- Bearings are quadrant strings: `N 45°30'00" E`, `S 12°15'30" W`, etc.
- All functions are pure (no I/O, no global state) and raise `ValueError` on geometrically undefined input (coincident points, parallel rays, non-intersecting circles).

## Testing

No test-runner dependency — stdlib `unittest` only:

```bash
python -m unittest discover -s tests
```

28 tests covering every public function, including hand-checked values (3-4-5 triangle intersections, closed square traverses, DMS round-trips).

## Project structure

```
survey-cogo/
├── src/cogo/
│   ├── __init__.py        # exports + __version__
│   ├── angles.py          # DMS, azimuth/bearing
│   ├── point.py           # Point model
│   ├── inverse.py         # inverse computation
│   ├── forward.py         # forward computation
│   ├── intersections.py   # bearing-bearing, distance-distance
│   ├── traverse.py        # traverse runs, misclosure, Bowditch
│   └── area.py            # polygon area / perimeter
├── tests/test_cogo.py
├── pyproject.toml
├── README.md
├── CHANGELOG.md
└── LICENSE
```

## Roadmap

`survey-cogo` is the foundation module. Planned siblings in the suite: `survey-levels` (leveling reduction), `survey-adjust` (least squares), `survey-geodesy` (datums/state plane), `survey-raster` (remote sensing), `survey-pointcloud` (LiDAR/DEM/volumes), `survey-gnss` (RINEX/baselines) — all tied together by a `survey-suite` desktop app.

## License

MIT — see [LICENSE](LICENSE).
