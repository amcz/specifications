# specifications
# CF-Compliant NetCDF Specification Guide

This document outlines the standards and conventions for creating [Climate and Forecast (CF)](http://cfconventions.org/) compliant NetCDF files for atmospheric and environmental data. The CF conventions aim to promote the processing and sharing of climate and forecast data using self-describing, portable NetCDF files.

---

## 📦 File Structure

A CF-compliant NetCDF file typically includes:

- **Dimensions** – named axes of the data (e.g., time, latitude, longitude, level).
- **Variables** – data arrays, each associated with a subset of the dimensions.
- **Attributes** – metadata for the file or individual variables (units, standard names, etc.).

---

## ✅ Required Global Attributes

```text
title               : Brief description of the dataset
institution         : Name of the institution that created the data
source              : How the data was generated (e.g., model name, observational platform)
history             : Audit trail of modifications (auto-generated)
references          : Citations or documentation
comment             : Additional information about the dataset
Conventions         : "CF-1.10"  # or the applicable version


# CF-Compliant NetCDF File Description

This document describes the structure and metadata of a CF-compliant NetCDF file with multiple dimensions including ensemble, vertical levels, and spatial coordinates. This file contains gridded atmospheric data, with bounds for time, space, and vertical coordinates.

---

## 📦 File Dimensions

```text
source: 1
ens: 1
time: 24
bnds: 2
y: 175
x: 3601
z: 12
```

---

## 🧭 Coordinate Variables

### `y (y)`

```text
Type: int64
Values: 1322 to 1496 (175 values)
```

- **Associated coordinate**: `latitude (y)`
  - `standard_name = "latitude"`
  - `units = "degrees_north"`

### `x (x)`

```text
Type: int64
Values: 1 to 3601
```

- **Associated coordinate**: `longitude (x)`
  - `standard_name = "longitude"`
  - `units = "degrees_east"`

### `bnds (bnds)`

```text
Type: int32
Values: 0, 1
```

Used to define bounds for coordinates.

### `latitude (y)`

```text
Type: float64
Values: 42.1 to 59.5
```

### `longitude (x)`

```text
Type: float64
Values: -180.0 to 180.0
```

### `z (z)`

```text
Type: float64
Values: 761.5 to 17520 (Pa or meters depending on context)
```

- Suggested metadata:
  - `standard_name = "air_pressure"` or "altitude"
  - `units = "Pa"` or "m"
  - `positive = "down"` (or "up")

### `time (time)`

```text
Type: datetime64[ns]
Values: 2020-10-21T20:45:00 to 2020-10-...
```

- Suggested metadata:
  - `standard_name = "time"`
  - `units = "hours since 2020-10-21 20:45:00"`
  - `calendar = "gregorian"`

### `ens (ens)`

```text
Type: <U2
Value: 's1'
```

- Ensemble member label, can be included as an auxiliary coordinate.

### `source (source)`

```text
Type: <U3
Value: 'gfs'
```

- Source model label, can also be an auxiliary coordinate.

---

## 📊 Data Variables

### `SUM (source, ens, time, z, y, x)`

```text
Type: float32
Values: Mostly 0.0s (sample shown)
```

- Suggested metadata:
  - `long_name = "Accumulated field"` (replace with actual meaning)
  - `units = "[appropriate units]"`
  - `standard_name = [use CF table if possible]`
  - `_FillValue = -9999.0` (or appropriate missing value)
  - `coordinates = "time z latitude longitude"`
  - `cell_methods = "time: sum"` (or other, depending on processing)

---

## 🔗 Bounds Variables

These define spatial and temporal bounds for CF compliance.

### `time_bounds (source, ens, time, bnds)`

### `latitude_bounds (source, ens, y, bnds)`

### `longitude_bounds (source, ens, x, bnds)`

### `z_bounds (source, ens, z, bnds)`

Each of these must be linked in the metadata of their corresponding coordinate variable:

```text
time:bounds = "time_bounds"
latitude:bounds = "latitude_bounds"
longitude:bounds = "longitude_bounds"
z:bounds = "z_bounds"
```

---

## 🗺️ CRS Variable

### `crs (source, ens)`

```text
Type: int32
Value: 0
```

- Include grid mapping metadata for spatial projection (if needed):
  ```text
  crs:grid_mapping_name = "latitude_longitude"
  crs:semi_major_axis = 6378137.0
  crs:inverse_flattening = 298.257223563
  crs:epsg_code = "EPSG:4326"
  ```

---

## ✅ Recommendations

- Validate with [CF Checker](https://github.com/cedadev/cf-checker)
- Ensure units are consistent and metadata is present
- Clarify role of `source` and `ens` as auxiliary coordinates or dimension labels
- Consider compression (`zlib=True`) and chunking for performance

---

## 📚 References

- [CF Conventions](http://cfconventions.org/)
- [Standard Name Table](http://cfconventions.org/standard-names.html)
- [NetCDF Format Guide](https://www.unidata.ucar.edu/software/netcdf/)
- [xarray Docs](https://docs.xarray.dev/)
- [NCO Tools](https://nco.sourceforge.net/)
