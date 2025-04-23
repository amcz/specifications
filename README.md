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
  Values: [0.2, 2, 5, 10]
  ATTRIBUTES
    units : mg m-3
    standard_name: volcanic_ash_air_concentration
    long_name: Threshold for exceedance probability
  ```

  *  see alternative methods of specifying vertical coordinate
  * different ways of specifying reference time for time coordinate may be used


## 📊Main Data Variable

### Concentration File

```text
ash_concentration (time, latitude, longitude, z)
ATTRIBUTES
   standard_name : mass_concentration_of_volcanic_ash_in_air
   long_name : volcanic ash mass concentration in air as determined from model
   units : mg m-3
   cell_methods : time: mean withinbounds, longitude: mean within bounds, latitude: mean within bounds, z: mean within bounds
```

* use appropriate description in cell methods
* mg/m^3 is also CF compliant unit specification

### Probabilistic File
```text
ash_probability (time, latitude, longitude, z, threshold)
ATTRIBUTES
   standard_name : probability_of_exceedance_of_volcanic_ash_air_concentration
   long_name : probability that volcanic ash concentration exceeds threshold as determined from model
   units : percent
```
* no appropriate standard_name exists with CF name tables for ash probability of exceedance.

## Flight levels

```text
flight_levels (z)
values [25, 75, 125, 175, 225, 275, 325, 375, 425, 475, 525, 575]
ATTRIBUTES
   standard_name : flight_level
   long_name : flight levels at center of vertical level
   bounds : flight_level_bounds
   units : 100 feet
   comment : flight level is defined as altitude in hundreds of feet
```
 * hft or hecta-feet is not considered compliant as it combines SI prefix with English unit
 * no standard_name in CF tables

## 🔗 Bounds Variables

These define spatial and temporal bounds for CF compliance.

```text
time_bounds (time bnds)

latitude_bounds (latitude, bnds)

longitude_bounds (longitude, bnds)

z_bounds (z, bnds)

flight_level_bounds (z, bnds)
```
* If not using temporal averaging then time_bounds does not need to be defined.
* z_bounds not needed if using alternative method of specifying vertical coordinate.
---

## 🗺️ Defining Mapping 

### `crs ()`

- Include grid mapping metadata for spatial projection:
- Below is example for spherical earth
  ```text
  ATTRIBUTES
      crs:grid_mapping_name = latitude_longitude
      earth_radius : 6378200.0
      long_name : Spherical earth with radius 6371.2 km
      comment : This grid uses spherical Earth approximation. No EPSG code applies.
  ```

---
## Alternative formulations

### Alternative specification for vertical coordinate

Some centers may use flight_level as the vertical coordinate and dimension in place of z. This may not work as well with some GIS software.

### Alternative method for latitude and longitude dimension

It is also CF compliant to define dimensions x, y instead of latitude, longitude.
Then the latitude and longitude coordinates have dimensions  latitude(y) and longitude(x).


## ✅ Checkers

- Validate with [CF Checker](https://github.com/cedadev/cf-checker)

---

## 📚 References

- [CF Conventions](http://cfconventions.org/)
- [Standard Name Table](http://cfconventions.org/standard-names.html)
- [NetCDF Format Guide](https://www.unidata.ucar.edu/software/netcdf/)
- [xarray Docs](https://docs.xarray.dev/)
- [NCO Tools](https://nco.sourceforge.net/)
