# specifications for QVA netcdf files
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
- [netcdf tutorial](#tutorial)
- [References](#-references)

---

## Summary of updates

#### 21 April 2026
* updated unknown value for volcano_id to be 6000000

#### 13 April 2026
* Added bnds coordinate variable back in. 

#### 6 April 2026
* fixed description of dimension sizes
* remove bnds coordinate variable and specify bounds variables by `X_bnds (X, 2)`
* provided more detail on specification of time unit
  

#### 25 March 2026
* removed alternative formulations
* created summary of differences and points that need to be decided

#### 13 March 2026
* Added specifications for London and Toulouse harmonized format.
* It was clarified that hft (hecta-feet) is a CF compliant unit
* Consequently using flight_level as a coordinate rather than data variable is recommended
* dimension order was clarified
* dimension order for probabilistic data is still under discussion


## Summary of Differences

- [global_attributes](#globa-attributes) Set of required attributes needs to be decided
- [event_type values](#event_type)  This needs to be resolved
- [grid centering](#grid_centering)
- [dimension ordering in probabilistic files](#dimension_order)
- [specification of reference time](#reference_time)
- Use of bnds coordinate variable
- Some VAACs may be using a string such as '999999' for volcano_id when volcano is unknown.
- Most VAACs use -180 to 180 for longitude values. Some VAACs use 0 to 360

## Other discussion points
What value should be used for volcano_id when volcano is unknown

VAACs should agree on a minimum concentration threshold (e.g., 0.01 mg/m³), below which all values are set to 0.

This avoids storing negligible values (e.g., 1e-10 mg/m³) in the file.

* Longitudes between -180 and 180.  vs 0 and 360.
    * If using WGS84 (EPSG:4326) to plot data, bounds are -180 to 180.
    * Most using -180 to 180.

## ✅Global Attributes

---
#### Global attributes providing basic metadata -  Required
```text
title               : Volcanic ash air concentration forecast
Conventions         : "CF-1.9"  # or other
institution         : name of originating institution
source              : VAAC [NAME] QVA   # for example VAAC LONDON QVA
history             : creation history (string)
volcano_id          : 300250 (string)
```
* volcano_id should be taken from the Smithsonian list https://volcano.si.edu/projects/vaac-data/.
* if volcano is unknown then use 600000. (see discussion below)
* Some VAACs use 999999 as volcano_id for tests and exercises
* history is a string that documents how the file was created. At the least, it should document the time of creation.

Discussions around what the volcano_id string should be if the volcano is unknown has included the following points.
* using an empty string
    * is consistent with IWXXM and some guidance 
    * may cause unwanted behavior in  the API
      ``` Users can query data for a specific volcano using their ID with a request URL of the form : {APIBaseURL}/collections/{collectionId}/locations/{locationId}
      Where locationId is the volcano_id. If it is empty, then this is the same as the request : {APIBaseURL}/collections/{collectionId}/locations/
      which returns information on all eruptions that currently have data. Therefore, it will not be possible to query data for an unknown volcano with an empty string in the URL.
      ```
  * 600000
     * Is already used by Toulouse and Tokyo for unknown volcano
  * 999999
     * Is used by London for unknown volcano but they could switch to 999999
     * Is used by some VAACs (Tokyo, Darwin) to indicate TEST or EXERCISE 



---
#### Global Attributes that indicate status - Required
```
event_type              : TEST, OPERATIONAL, REAL EVENT, EXERCISE
report_status            : NORMAL, CORRECTION
permissible_usage        : NON_OPERATIONAL, OPERATIONAL
permissible_usage_reason  : TEST, EXERCISE
remarks                  : string
```
* remarks may be a generic message if workflow does not allow for adding remarks.

#### Global attributes that indicate status - Optional
```
event_type              : TEST, OPERATIONAL (REAL EVENT), EXERCISE
report_status           : NORMAL, CORRECTION
reportStatus_definitions  : NORMAL: first issuance; CORRECTION: correction to previous issuance
permissibleUsage_definitions   : OPERATIONAL: data may be used for operational purposes, NON-OPERATIONAL: Data should not be used for operational purposes but may be used for other purposes
permissibleUsageReason_definitions  : EXERCISE: produced for an exercise, TEST: produced for a test, HYPOTHETICAL: produced for possible future event
```

#### Discussion on event_type
`event_type` is used in the API for filtering. `permissibleUsage` and `permissibleUsageReason` are in the IWXXM schema. 
The table below gives the relationship between the `event_type` attributes and the `permissable_usage` and `permissable_usage_reason` attributes.
Note that ```OPERATIONAL``` and ```REAL EVENT``` have the same meaning.
Some VAACs are using ```OPERATIONAL``` to be consistent with the permissable_usage field while others use ```REAL EVENT``` in
response to user feedback that this is a better description.

## Rules

| Condition                     | `permissibleUsage`   | `permissibleUsageReason` |
|------------------------------|----------------------|---------------------------|
| `event_type == TEST`         | NON_OPERATIONAL      | TEST                      |
| `event_type == EXERCISE`     | NON_OPERATIONAL      | EXERCISE                  |
| `event_type == OPERATIONAL or REAL EVENT`   | OPERATIONAL          |  does not exist in file |


<a id='event_type'></a>
DISCUSSION
* for the `event_type` we should  agree to use either 'REAL EVENT' or 'OPERATIONAL' as this will be used to filter.
* `event_type` is redundant with `permissibleUsage` and `permissibleUsageReason`
* Including all three allows to be consistent with the IWXXM schema and API. 

---


#### Global attributes providing extra metadata- Recommended

```

reference           : URL for insitution / model
volcano_name         : Bezymianny
release_location    : lat: XX.XXXN, lon: XX.XXXW
meteorological_data : GFSQ  # short identifier for NWP data utilized
WMO_category        : Volcanic Ash
WMO_originator      : KNES   # unique VAAC identifier
product_type        : volcanic ash forecast
issue_time          : when forecasters issued the data (in iso 8601 format, e.g. 2026-03-04T13:34:00Z)

```
* volcano names should be taken from the Smithsonian list https://volcano.si.edu/projects/vaac-data/.
* if volcano is unknown then 'unknown' should be used for volcano_name
* WMO_originator is a unique identifier for each VAAC. (e.g. KNES refers to Washington VAAC). 

It is recommended that volcano_id be the primary way of identifying volcanic source as volcano names may
have different spellings or contain special characters.

#### Extra global attributes may be present

```

```

## 📦Dimensions


### Concentration File 
```text
bnds: 2
time: unlimited
flight_level: unlimited   
latitude: unlimited
longitude: unlimited
```

### Probabilistic File
```text
threshold : Nth
bnds : 2
time: Nt
flight_level : Nz   
latitude: Ny
longitude: Nx
```
Dimension sizes are generally fixed within a given file but may vary between files.
No dimension is required to be defined as unlimited.

* For the base service level, the `flight_level` dimension has a size of 12. However, it is recommended that software be designed to handle varying numbers and values of flight levels, to accommodate future changes and evolution of the service.
* For the base service level, `time` dimension has a size of 9. However, it is recommended that software be designed to handle varying number of time values to accommodate future changes and evolution of the service. 
* For the base service level, the `threshold` dimension has a size of 4. However it is recommended that software be designed to handle varying numbers and values of threshold to accomodate future changes and evolution of the service.
* for the base service level, `latitude` and `longitude` dimensions will have variable sizes.


### Dimension order
<a id="dimension_order"></a>
We recommend that software be sensible to dimension order as it may be different across VAACs.

#### Recommended Dimension Order for NetCDF Variables according to CF convention

According to the CF convention, the recommended order for spatiotemporal dimensions is:

1. **T** – time
2. **Z** – flight_level
3. **Y** – Latitude
4. **X** – Longitude

All other dimensions (e.g., `threshold` etc.) should, whenever possible, be placed **to the left** of the spatiotemporal dimensions.
However using time as a first dimension is expected to be more efficient if the API requests tend to retrieve data by time.
Consequently some VAACs have opted to use time as the first dimension.

## Recommended Dimension Orders for NetCDF Variables

| VAAC | Data Variable | Order of Dimensions | Notes |
|------|---------------|---------------------|-------|
| Toulouse | ash_probability | threshold, time, flight_level, latitude, longitude | Follows CF convention recommendation to put non-spatiotemporal dimensions first. |
| London   | ash_probability | time, threshold, flight_level, latitude, longitude | Optimized for API requests that primarily access data by time. |
| All      | ash_concentration | time, flight_level, latitude, longitude |   |

---

## 🧭Coordinate Variables
and their associated dimensions

###  Concentration File Coordinate Variables
```text
latitude(latitude)
ATTRIBUTES
   standard_name : latitude
   units :  degrees_north
   bounds : latitude_bounds
   long_name : latitude degrees north from the equator
   axis : Y 

longitude(longitude)
ATTRIBUTES
   standard_name : longitude
   units :  degrees_east
   bounds : longitude_bounds
   long_name :  longitude degrees east from the greenwich meridian
   axis : X

time(time)
ATTRIBUTES
  units : hours since YYYY-MM-DD HH:MM:SSZ
  long_name: time at beginning of sampling period
  standard_name : time
  calendar : standard OR gregorian
  bounds : time_bounds
  axis : T

flight_level(flight_level) ;
base service values [25, 75, 125, 175, 225, 275, 325, 375, 425, 475, 525, 575]
ATTRIBUTES
		long_name       : flight level 
		units           : hectofeet (or hectoft, hft, etc...)
		axis            : Z 
		positive        : up 
		reference_datum : sea level pressure datum of 1013.25 hPa 
		bounds          : flight_level_bounds

bnds(bnds)
TYPE: int32
values: 0,1

```
For the time variable, any CF compliant unit may be utilized. Other examples include

* `units: minutes since YYYY-MM-DD HH:MM:SSZ`
* `units: seconds since YYYY-MM-DD HH:MM:SSZ`

## Probabilistic File Coordinate variables

Same coordinates as concentration file but with one additional coordinate.

  ```text
  threshold
  Values: [0.2, 2, 5, 10]
  ATTRIBUTES
    units : mg m-3  OR mg/m^3
    standard_name: volcanic_ash_air_concentration
    long_name: Threshold for exceedance probability
  ```

<a id="reference_time"></a>
*  different ways of specifying reference time for time coordinate may be used
* The base service standard resolution of the spatial grid is 0.25 degrees but higher resolutions may be implemented.
<a id='grid_centering'></a>
* Currently some VAACs center their grids on X.0 while others center their grids on X.125.
While it is preferential that VAACs define their grids so they can be easily overlapped, it is not possible
currently for all VAACs to change their operational setup to allow this. However it seems that VAACs who have more
frequent hand-offs do have similar setups. 


## 📊Main Data Variable

### Concentration File Data Variable

```text
ash_concentration (time, flight_level, latitude, longitude)  # London and Toulouse
ATTRIBUTES
   standard_name : mass_concentration_of_volcanic_ash_in_air
   long_name : volcanic ash mass concentration in air as determined from model
   units : mg m-3
   cell_methods : time: mean withinbounds, longitude: mean within bounds, latitude: mean within bounds, z: mean within bounds
```

* use appropriate description in cell methods
* mg/m^3 is also CF compliant unit specification

### Probabilistic File Data Variable
```text
ash_probability (threshold, time, flight_level, latitude, longitude, # Toulouse
ash_probability (time, threshold, flight_level, latitude, longitude) # London
ATTRIBUTES
   standard_name : probability_of_exceedance_of_volcanic_ash_air_concentration
   long_name : probability that volcanic ash concentration exceeds threshold as determined from model
   units : percent
```
* no appropriate standard_name exists with CF name tables for ash probability of exceedance.
* Toulouse and London have differing dimension orders. 


## Bounds Variables 

These define spatial and temporal bounds for CF compliance. They are the same for the probabilistic and concentration files.

```text
time_bounds (time, bnds)

latitude_bounds (latitude, bnds)

longitude_bounds (longitude, bnds)

flight_level_bounds (flight_level, bnds)
```
* If not using temporal averaging then time_bounds does not need to be defined.

A bnds coordinate of size 2 may be defined.
Alternatively the bnds coordinate variable may be omitted and bounds can be specified simply with 
```
time_bounds (time, 2)
```

Some examples are provided here
https://cf-convention.github.io/Data/cf-conventions/cf-conventions-1.13/cf-conventions.html#bounds-one-d

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
## NetCDF File Structure (Simple Overview)
<a id='tutorial'></a>

A NetCDF file is a self-describing, binary format used to store scientific data. It contains both the data and the metadata needed to understand it.

### Main Components

A NetCDF file is made up of three main parts:

#### 1. Dimensions
Dimensions define the size and shape of the data.

Examples:
- `time` (e.g., 10 steps)
- `latitude` (e.g., 180 points)
- `longitude` (e.g., 360 points)
- `flight_level` (e.g., 12 levels)

---

#### 2. Variables
Variables store the actual data and coordinate information.

There are two types:

- **Coordinate variables**: define the axes (e.g., `time`, `latitude`, `longitude`)
- **Data variables**: contain the measured or modeled data (e.g., `ash_probability`)


---

#### 3. Attributes
Attributes provide metadata (information about the data).
* Attributes can be associated with the entire file (global attributes)
* Attributes can also be associated with individual data variables (variable attributes)
They can describe:
- Units (e.g., `mg/m³`)
- Variable meaning (`long_name`)
- File-level information (e.g., title, source)

---

#### 4. Bounds (`bnds`) Variables

Bounds variables define the edges (limits) of coordinate values.

They are used when coordinates represent **intervals** rather than single points.  
For example, a time value may represent a period (start–end), or a latitude may represent a grid cell.

### Key Points

- Typically named `<coordinate>_bnds` (e.g., `time_bnds`, `latitude_bnds`)
- Have one extra dimension (usually `bnds = 2`) for lower and upper limits
- Linked to a coordinate variable using the `bounds` attribute
---

### How It Fits Together

- **Dimensions** define the grid
- **Coordinate variables** give meaning to each axis
- **Data variables** store values on that grid
- **Attributes** describe the data

Together, these make the file **self-describing**, so users and software can interpret the data without external documentation.


---
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
