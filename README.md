# specifications
# CF-Compliant NetCDF Specification Guide

This document outlines the standards and conventions for creating [Climate and Forecast (CF)](http://cfconventions.org/) compliant NetCDF files for volcanic ash forecasts. The CF conventions aim to promote the processing and sharing of climate and forecast data using self-describing, portable NetCDF files. Two file types are described. One contains information on ash concentrations while the other contains information on probabilities of exceedances of ash concentrations.

## Table of Contents
- [Global Attributes](#global-attributes)
- [Dimensions](#file-dimensions)
- [Coordinate Variables](#coordinate-variables)
- [Main Data Variable](#main-data-variable)
- [Flight Levels](#flight-levels)
- [Bounds Variables](#bounds-variables)
- [Defining Mapping](#defining-mapping)
- [Alternative Formulations](#alternative-formulations)
- [References](#-references)

---


## ✅Global Attributes

---
Required
```text
title               : Volcanic ash air concentration forecast
Conventions         : "CF-1.9"  # or other
institution         : name of originating institution
source              : VAAC [NAME] QVA   # for example VAAC LONDON QVA
history             : date created
volcano_id          : 300250
```
* volcano id (vid) should be taken from the Smithsonian list https://volcano.si.edu/projects/vaac-data/.
* if volcano is unknown then 'none' should be used
  

---
Required
```
reportStatus            : NORMAL
permissableUsage        : NON-OPERATIONAL
permissableUsageReason  : TEST
reportStatus_definitions  : NORMAL: first issuance; CORRECTION: correction to previous issuance
permissableUsage_definitions   : OPERATIONAL: data may be used for operational purposes, NON-OPERATIONAL: Data should not be used for operational purposes but may be used for other purposes
permissableUsageReason_definitions  : EXERCISE: produced for an exercise, TEST: produced for a test, HYPOTHETICAL: produced for possible future event
remarks                  :
```

* reportStatus, permissableUsage, and permissableUsageReason are fields in the IWXXM file which should be included in the netcdf so end users receive the same information.

* remarks have also been added to iwxxm and should be available in netcdf.

**points for discussion**
London and Toulouse VAAC propose using event_type to  replace PermissibleUsage and PermissibleUsageReason that are used in IWXXM files
```
event_type : TEST      # REAL EVENT (or OPERATIONAL), TEST, EXERCISE are allowed
```

---
Recommended

```

reference           : URL for insitution / model
volcanoname         : Bezymianny
release_location    : lat: XX.XXXN, lon: XX.XXXW
meteorological_data : GFSQ  # short identifier for NWP data utilized
WMO_category        : Volcanic Ash
WMO_originator      : KNES   # unique VAAC identifier
product_type        : volcanic ash forecast

```

* volcano names should be taken from the Smithsonian list https://volcano.si.edu/projects/vaac-data/.
* if volcano is unknown then 'unknown' should be used for volcano_name
* WMO_originator is a unique identifier for each VAAC

Other global attributes could potentially be added.

## 📦Dimensions


### Concentration File 
```text
bnds: 2
time: unlimited
z: 12    # alternative flight_levels
latitude: unlimited
longitude: unlimited
```

### Probabilistic File
```text
threshold : 4
bnds : 2
time: unlimited
z : 12   # alternative flight_levels
latitude: unlimited
longitude: unlimited
```





---

## 🧭Coordinate Variables
and their associated dimensions

###  Concentration File
```text
latitude (latitude)
ATTRIBUTES
   standard_name : latitude
   units :  degrees_north
   bounds : latitude_bounds
   long_name : latitude degrees north from the equator
   axis : Y 

longitude (longitude)
ATTRIBUTES
   standard_name : longitude
   units :  degrees_east
   bounds : longitude_bounds
   long_name :  longitude degrees east from the greenwich meridian
   axis : X

z(z)
ATTRIBUTES
  units : feet
  standard_name : height_above_mean_sea_level
  positive : up
  bounds : z_bounds
  long_name : altitude at center of vertical level
  axis : Z

time(time)
ATTRIBUTES
  units : hours since YYYY-MM-DD HH:MM:SSZ
  long_name: time at beginning of sampling period
  standard_name : time
  calendar : standard OR gregorian
  bounds : time_bounds
  axis : T

bnds (bnds)
Type: int32
Values: 0,1

```

## Horizontal grid resolution and specification

* The standard resolution of the spatial grid is 0.25 degrees
* Higher resolutions may be allowed
* Currently some VAACs center their grids on X.0 while others center their grids on X.125.
While it is preferential that VAACs define their grids so they can be easily overlapped, it is not possible
currently for all VAACs to change their operational setup to allow this. However it seems that VAACs who have more
frequent hand-offs do have similar setups. 

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
  *  mg/m^3 may also be used
  *  different ways of specifying reference time for time coordinate may be used


## 📊Main Data Variable

### Concentration File

```text
ash_concentration (time, z, latitude, longitude)
ash_concentration (time, flight_level, latitude, longitude)  # London and Toulouse
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
ash_probability (threshold, time, z, latitude, longitude)  # Generic
ash_probability (threshold, time, flight_level, latitude, longitude, # Toulouse
ash_probability (time, threshold, flight_level, latitude, longitude) # London
ATTRIBUTES
   standard_name : probability_of_exceedance_of_volcanic_ash_air_concentration
   long_name : probability that volcanic ash concentration exceeds threshold as determined from model
   units : percent
```
* no appropriate standard_name exists with CF name tables for ash probability of exceedance.
* Toulouse and London have differing dimension orders. 

### Dimension order

The order of the dimensions can be important for some software packages that ingest the netcdf.
From the CF convention, it is recommanded to use threshold as first dimension (VAAC Toulouse choice): « If any or all of the dimensions of a variable have the interpretations of "date or time" (T), "height or depth" (Z), "latitude" (Y), or "longitude" (X) then we recommend, but do not require (see Section 1.5, "Relationship to the COARDS Conventions"), those dimensions to appear in the relative order T, then Z, then Y, then X in the CDL definition corresponding to the file. All other dimensions should, whenever possible, be placed to the left of the spatiotemporal dimensions.»

But, using time as first dimension is also expected to be more efficient if API requests tend to retrieve data by time (VAAC London choice).

A similar discussion can be found in cf-convention discussion repository where keeping the recommended order is suggested as well as to using chunks in netcdf to match access patterns.

VAAC London and VAAC Toulouse are both reluctant to change, to avoid a step back later, but highlight that difference in their documentations for end users.

## Flight Levels

Data variable which provides vertical levels in FL 

```text
flight_level (z)
values [25, 75, 125, 175, 225, 275, 325, 375, 425, 475, 525, 575]
ATTRIBUTES
   standard_name : flight_level
   long_name : flight level at center of vertical level
   bounds : flight_level_bounds
   units : 100 feet
   comment : flight level is defined as altitude in hundreds of feet
```
 * hft or hecta-feet is not considered compliant as it combines SI prefix with English unit
 * no standard_name in CF tables
 * this data variable may be ommitted if flight_levels is utilized as a dimension/coordinate in the alternative formulation.
 * 07/31/2025 changed from flight_levels to flight_level

## Bounds Variables 

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

## Defining Mapping 

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

Some centers may use flight_level as the vertical coordinate and dimension in place of z. This may not work as well with some GIS software which may not recognize 100 ft as a CF compliant unit.

### Alternative method for latitude and longitude dimension

It is also CF compliant to define dimensions x, y instead of latitude, longitude.
Then the latitude and longitude coordinates have dimensions  latitude(y) and longitude(x).


## ✅ Checkers

- Validate with [CF Checker](https://github.com/cedadev/cf-checker)

---

## 📚 References

- [CF Conventions](http://cfconventions.org/)
- [Standard Name Table](http://cfconventions.org/standard-names.html)
- [CF compliant units](https://www.unidata.ucar.edu/software/udunits/)
- [NetCDF Format Guide](https://www.unidata.ucar.edu/software/netcdf/)
- [xarray Docs](https://docs.xarray.dev/)
- [Volcano Names and IDs](https://volcano.si.edu/projects/vaac-data/)
