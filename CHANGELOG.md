# Changelog

All notable changes to `survey-cogo`. Follows semantic versioning.

## [0.1.0] - 2026-09-23
### Added
- Initial release: pure-Python, dependency-free COGO engine.
- `angles`: DMS/decimal conversion and parsing, azimuth/bearing conversion.
- `inverse` / `forward`: azimuth-distance between points and point by direction.
- `intersections`: bearing-bearing and distance-distance (two solutions).
- `traverse`: turned-angle to azimuth conversion, open/closed traverse runs,
  angular and linear misclosure, relative precision, Compass (Bowditch) adjustment.
- `area`: polygon area by coordinates, perimeter.
- 28 unit tests (stdlib `unittest`, no test-runner dependency).
