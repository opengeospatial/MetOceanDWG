# Parametric datum for pressure measurements

This page describe a more exploratory work compared to the proposal on coordinate operations.

## Problem statement

_Note: the following discussion with vertical datum is only for illustrating the problem. We are not
suggesting to use them for pressure measurements._

For vertical height measurement, the EPSG database provide datum for various surfaces
from which measurements can be done. For example:

```
VerticalDatum["Low Water", Id["EPSG", 1093, URI["urn:ogc:def:datum:EPSG::1093"]]]
VerticalDatum["High Water", Id["EPSG", 1094, URI["urn:ogc:def:datum:EPSG::1094"]]]
VerticalDatum["Lower Low Water Large Tide", Id["EPSG", 1083, URI[ …omitted… ]]]
```

Above datum allow the construction of vertical CRS with meter units.
We do not need to provide full CRS definitions;
data providers can assemble axis direction (up or down) and linear unit of their choice with above datum.
But the datum identification is significant.

**Use case:** if a software wants to report depths on a nautical chart for navigation purpose,
it shall show depths at the lowest possible tide.
The software may need to recognize “Lower Low Water Large Tide” as the desired datum
and rejects depths expressed with other datum (if we don't apply any datum shift).
Parsing the datum name as a text is not reliable. Checking the authority datum code
is a more machine-readable way to check if the datum is okay for reporting depths on a nautical chart.
Above works for heights measured in meters,
but there is no authority codes for identifying parametric datum for pressure measurements.


### Questions

[ISO 19162 §12.4](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html#97) gives the following example:

```
PARAMETRICCRS["WMO standard atmosphere layer 0",
  PDATUM["Mean Sea Level", ANCHOR["1013.25 hPa at 15°C"]],
  CS[parametric, 1],
  AXIS["pressure (hPa)", up],
  PARAMETRICUNIT["HectoPascal", 100.0]]
```

Do we have WMO or UCAR codes for specifying the surface from which pressure measurements are done
(_“Mean Sea Level”_ in above example)?
The reason is similar to the _“Lower Low Water Large Tide”_ example:
a software may accept only data relative to some expected datum.

Grib2 code table 4.5 — Fixed Surface Types and Units — defines the following entries:

> Code: 108
> Meaning: Level at Specified Pressure Difference from Ground to Level
> Unit: Pa

Is it suitable for defining the datum of coordinates measured as (pressured on the ground)
− (pressure at the measured location)? Is the following correct?

```
ParametricCRS[“Elevation as pressure difference”,
  ParametricDatum["Level at Specified Pressure Difference from Ground to Level",
    ID["WMO", 108, URI["urn:ogc:def:datum:WMO::108"]]],
  CS[parametric, 1],
  AXIS["pressure (Pa)", up],
  PARAMETRICUNIT["Pascal", 1.0]]]
```

Questions:

* Which WMO tables and codes can be used as datum definitions
  (in addition of table 4.5 code 108 shown above, if correct).
* `URI["urn:ogc:def:datum:WMO::108"]` does not include table number.
  Does WMO has codes that are unique across Grib tables?
* What does _“Axis direction: up”_ means in a context where coordinate values are increasing downward
  (case of pressure)? ISO 19162 said: _“axis positive direction is up relative to gravity”_.
  Is _“axis positive direction”_ synonymous of _“direction of increasing values”_?
