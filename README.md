# specifications
# CF-Compliant NetCDF Specification Guide

This document outlines the standards and conventions for creating [Climate and Forecast (CF)](http://cfconventions.org/) compliant NetCDF files for atmospheric and environmental data. The CF conventions aim to promote the processing and sharing of climate and forecast data using self-describing, portable NetCDF files.

- [Global Attributes](#global-attributes)
- [File Dimensions](#file-dimensions)
- [Coordinate Variables](#coordinate-variables)
- [Alternative Formulations](#alternative-formulations)
---

## 📦 File Structure

A CF-compliant NetCDF file typically includes:

- **Dimensions** – named axes of the data (e.g., time, latitude, longitude, level).
- **Variables** – data arrays, each associated with a subset of the dimensions.
- **Attributes** – metadata for the file or individual variables (units, standard names, etc.).

---

## ✅ Global Attributes

```text
title               : Volcanic ash air concentration forecast
status              : NORMAL
usage               : NON-OPERATIONAL
reason              : test
source              : HYSPLIT model
institution         : XXXX
reference           : URL for insitution / model
history             : Audit trail of modifications (auto-generated)
volcanoname         : Bezymianny (or unknown)
vid                 : 300250
release_location    : lat: XX.XXXN, lon: XX.XXXW
meteorological_data : GFSQ
Conventions         : "CF-1.9"  # or the applicable version
WMO_category        : Volcanic Ash
WMO_originator      : KNES
status_definitions  : 
usage_definitions   :
reason_definitions  :
product_type        : volcanic ash forecast
```

* volcano names and volcano id (vid) should be taken from the Smithsonian list.

## 📦 File Dimensions

### Concentration File 
```text
time: unlimited
bnds: 2
latitude: unlimited
longitude: unlimited
z: 12
```

### Probabilistic File
```text
time: unlimited
bnds: 2
latitude: unlimited
longitude: unlimited
z: 12
threshold: 4
```


---

## 🧭 Coordinate Variables and their associated dimensions

###  Concentration File
```text
latitude (longitude)
ATTRIBUTES
   standard_name : latitude
   units :  degrees_north
   bounds : latitude_bounds

longitude (longitude)
ATTRIBUTES
   standard_name : latitude
   units :  degrees_north
   bounds : latitude_bounds

bnds (bnds)
Type: int32
Values: 0,1

z(z)
ATTRIBUTES
  units : feet
  standard_name : height_above_mean_sea_level
  positive : up
  bounds : z_bounds
  long_name : altitude at center of vertical level

time(time)
ATTRIBUTES
  units : hours since YYYY-MM-DD HH:MM:SSZ
  long_name: time at beginning of sampling period
  standard_name : time
  calendar : standard
  bounds : time_bounds
```

## Probabilistic File

Same coordinates as concentration file but with one additional coordinate.

  ```text
  threshold
  Values: 0.2 2 5 10
  ATTRIBUTES
    units : mg m-3
    standard_name: volcanic_ash_air_concentration
    long_name: Threshold for exceedance probability
  ```

  *  see alternative methods of specifying vertical coordinate
  * different ways of specifying reference time for time coordinate may be used


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
## Alternative formulations


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
