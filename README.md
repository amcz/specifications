# CF-Compliant NetCDF Specification Guide

> There is a need to try to align to common conventions for **end users**, especially
> between VAACs that act as backup to each other. Particularily :
>
> - Dimension names
> - Variable names
> - Units
> - Dimensions order within concentration and probability arrays (dimensions order can be important for some softwares)
> - Some relevant attributes - **not all !!** 

This document outlines the standards and conventions for creating [Climate and Forecast (CF)](http://cfconventions.org/) compliant NetCDF files for volcanic ash forecasts. The CF conventions aim to promote the processing and sharing of climate and forecast data using self-describing, portable NetCDF files. Two file types are described. One contains information on ash concentrations while the other contains information on probabilities of exceedances of ash concentrations.


# Global Attributes

```yaml
Conventions: "CF-1.9" # or another version 
title      : Volcanic ash air concentration forecast
institution: name of originating institution
history    : date created
source     : VAAC [NAME] QVA   # for example VAAC LONDON QVA
event_type : OPERATIONAL
volcano_id : "300250"
```

* volcano id (volcano_id) should be taken from the [Smithsonian list](https://volcano.si.edu/projects/vaac-data/)
* if volcano is unknown then 'none' should be used for volcano_id
* event_type replace usage, and reason that are utilized in IWXXM file
* history is needed for CF compliance. Unclear what information it should contain.

Other global attributes **can** be added, as an  example :  

```yaml
issue_time         : when forecaters issued the data 
usage              : OPERATIONAL/NON-OPERATIONAL
reason             : none, EXERCISE or TEST (none when OPERATIONAL)
run_time           : runtime of the model (when it was launched)
reference          : URL for insitution / model
volcano_name       : Bezymianny
release_location   : "lat: XX.XXXN, lon: XX.XXXW"
meteorological_data: GFSQ
comments           :
# others attributes...
# Some attributes may be necessary for some VAAC operational setups.
```

# Dimensions

## Concentration File 

### Formulation for VAAC London and VAAC Toulouse

```yaml
time        : TTTTT
bnds        : 2
latitude    : YYYYY
longitude   : XXXXX
flight_level: 12
```

> **grid dimension**: the grid dimensions (number of points along longitude and latitude) can be different between VAAC because the grid can be global or local (clipped around the ash cloud to save storage)

Dimensions **time**, **longitude** and **latitude** can be unlimited, or not.
 
### Alternative formulation :

```diff
time: TTTTT
bnds: 2
latitude: YYYYY
longitude: XXXXX
+ z: 12
- flight_level: 12
```

## Probabilistic File

A dimension for the number of thresholds is added.

### Formulation for VAAC London and VAAC Toulouse

```yaml
time: TTTTT
bnds: 2
latitude: YYYYY
longitude: XXXXX
flight_level: 12
threshold: 4
```

### Alternative formulation :

```diff
time: TTTTT
bnds: 2
latitude: YYYYY
longitude: XXXXX
- flight_level: 12
+ z: 12
threshold: 4
```

# Coordinate Variables

## Horizontal grib - longitude and latitude

* The standard resolution of the spatial grid is 0.25 degrees
* Higher resolutions may be allowed
* Currently some VAACs center their grids on X.0 while others center their grids on X.125.
While it is preferential that VAACs define their grids so they can be easily overlapped, it is not possible
currently for all VAACs to change their operational setup to allow this. 

```text
longitude (longitude)
ATTRIBUTES
   standard_name : longitude
   units :  degrees_east
   bounds : longitude_bounds
   long_name :  longitude degrees east from the greenwich meridian
   axis : X

longitude_bounds (longitude, bnds)

latitude (latitude)
ATTRIBUTES
   standard_name : latitude
   units :  degrees_north
   bounds : latitude_bounds
   long_name : latitude degrees north from the equator
   axis : Y 

latitude_bounds (latitude, bnds)
```

## Vertical coordinates

Flight Levels correspond level in an ICAO Standard atmosphere : they follow isobaric levels.

### Formulation for VAAC London and VAAC Toulouse

```text
flight_level(flight_level) ;
values [25, 75, 125, 175, 225, 275, 325, 375, 425, 475, 525, 575]
ATTRIBUTES
		long_name       : flight level 
		units           : hetctofeet (or hectoft, hft, 100 ft, etc...)
		axis            : Z 
		positive        : up 
		reference_datum : sea level pressure datum of 1013.25 hPa 
		bounds          : flight_level_bounds 

flight_level_bounds(flight_level, bnds)
```

> **reference_datum** is used to highlight the fact that Flight Level correspond to heights above sea level in
> an ICAO Standard Atmosphere (doc 7488)

* no standard_name in CF tables

* Levels are the middle of the 50-FL layer

### Alternative formulation :

```diff
- flight_level(flight_level) ;
+ z(z)
ATTRIBUTES
- units        : 100 ft (or hectoft, hft, etc...)
+ units        : feet
  standard_name: height_above_mean_sea_level
  positive     : up
- bounds       : flight_level_bounds 
+ bounds       : z_bounds
  long_name    : altitude at center of vertical level
  axis         : Z

- flight_level_bounds(flight_level, bnds)
+ z_bounds(z, bnds)
```

**QUESTION** : add a reference_datum as for the previous formulation ?

A data variable which provides vertical levels in FL is also added : flight_level is not a coordinate variable in that case.

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

## Time

```text
time(time)
ATTRIBUTES
  units : hours/minutes since YYYY-MM-DD HH:MM:SSZ
  long_name: time
  standard_name : time
  calendar : standard OR gregorian
  bounds : time_bounds   # Optional
  axis : T   # optional, depends on the dimensions order for probabilitites

time_bounds (time, bnds)
```

- If not using temporal averaging then time_bounds does not need to be defined.
- Time is the begining of the sampling period (if there is a sampling period)
- If time is not the 4th axis in variable ash concentration or ash probability, `axis: T` shouldn't be added.

## Probabilistic File

Same coordinate variables as concentration file but with one additional coordinate variable for thresholds.

```text
threshold
Values: [0.2, 2.0, 5.0, 10.0]
ATTRIBUTES
  units : mg m-3 (or mg/m^3, mg m**-3, etc...)
  standard_name: mass_concentration_of_volcanic_ash_in_air
  long_name: Threshold for exceedance probability
```

# Main Data Variable

## Concentration File

```text
ash_concentration (time, flight_level or z, latitude, longitude)
ATTRIBUTES
   standard_name : mass_concentration_of_volcanic_ash_in_air
   long_name : mass_concentration_of_volcanic_ash_in_air
   units : mg m-3 (or mg/m^3, ...)
   cell_methods : flight_level (or z): mean
or cell_methods : time: mean flight_level (or z): mean latitude: mean longitude: mean 
```

* use appropriate description in cell methods
* depending of the dimension choosen (VAAC London/Toulouse or alternative), choose either flight_level or z as 3rd axis.

## Probabilistic File

### VAAC London

```text
ash_probability (threshold, time, flight_level or z, latitude, longitude)
ATTRIBUTES
   long_name : probability_of_mass_concentration_of_volcanic_ash_in_air_above_threshold
   units : percent
```

### VAAC Toulouse

```diff
+ ash_probability (threshold, time, flight_level or z, latitude, longitude)
- ash_probability (time, threshold, flight_level or z, latitude, longitude)
ATTRIBUTES
   long_name : probability_of_mass_concentration_of_volcanic_ash_in_air_above_threshold
   units : percent
```

* no appropriate standard_name exists with CF name tables for ash probability of exceedance.

* depending of the dimension choosen for level (VAAC London/Toulouse or alternative), choose either flight_level or z as 3rd axis

#### Discussion on dimension order (threshold or time as first axis) :

From the CF convention, it is recommanded to use threshold as first dimension (VAAC Toulouse choice): 
« If any or all of the dimensions of a variable have the interpretations of "date or time" (T), "height or depth" (Z), "latitude" (Y), or "longitude" (X) then we **recommend, but do not require** (see Section 1.5, "Relationship to the COARDS Conventions"), those dimensions to appear in the relative order T, then Z, then Y, then X in the CDL definition corresponding to the file. All other dimensions should, **whenever possible**, be placed to the left of the spatiotemporal dimensions.»

But, using time as first dimension is also expected to be more efficient if API requests tend to retrieve data by time (VAAC London choice).

A similar discussion can be found in [cf-convention discussion repository](https://github.com/orgs/cf-convention/discussions/423) where keeping the recommended order is suggested as well as to using chunks in netcdf to match access patterns.

> VAAC London and VAAC Toulouse are both reluctant to change, to avoid a step back later, but highlight that difference in their documentations for end users. 

# Defining Mapping 

Optional : not used by VAAC London or VAAC Toulouse at the moment.

## `crs ()`

- Include grid mapping metadata for spatial projection:
- Below is example for spherical earth
  ```text
  ATTRIBUTES
      crs:grid_mapping_name = latitude_longitude
      earth_radius : 6378200.0
      long_name : Spherical earth with radius 6371.2 km
      comment : This grid uses spherical Earth approximation. No EPSG code applies.
  ```

# Links

## ✅ Checkers

- [CF Checker](https://github.com/cedadev/cf-checker)
- [IOOS Compliance Checker](https://github.com/ioos/compliance-checker) 
- [CF conformance Document](https://cfconventions.org/cf-conventions/conformance.html)

## 📚 References

- [CF Conventions](http://cfconventions.org/)
- [Standard Name Table](http://cfconventions.org/standard-names.html)
- [CF compliant units](https://www.unidata.ucar.edu/software/udunits/)
- [NetCDF Format Guide](https://www.unidata.ucar.edu/software/netcdf/)
- [Sofware that "understands" CF Data](https://cfconventions.org/software.html)
- [xarray Docs](https://docs.xarray.dev/)
- [Volcano Names and IDs](https://volcano.si.edu/projects/vaac-data/)
