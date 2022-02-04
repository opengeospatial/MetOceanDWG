# OGC as a source of authority codes for pole rotation

The preferred way to define the pole rotation method would be to
leave the task to expert in this field, [such as WMO](WMO.md).
As a fallback, this page provides a proposal for OGC definition.
We try to combine the example provided in
[ISO 19162 §14.3.2](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html#116) together with
[Rotated pole in CF-conventions](https://cfconventions.org/cf-conventions/cf-conventions.html#_rotated_pole)
and WMO Manual on Codes template 3.1.

There is two operation methods, depending if the rotation is applied on the north pole or the south pole.
For each operation, the name used in the UCAR netCDF library are shown.
All proposals below are identical to ISO 19162 §14.3.2 example
except for "North" or "South" prefix before operation name.




## South pole rotation

| Authority                 | Name or identifier            |
| ------------------------- | ----------------------------- |
| OGC identifier (proposal) | `urn:ogc:def:method:OGC::100` |
| OGC name (proposal)       | `South pole rotation`         |
| WMO name                  | `Rotated Latitude/longitude`  |
| Name in UCAR library      | `rotated_latlon_grib`         |


### Parameters

| Identifier proposal              | Name proposal             | Name in UCAR library        |
| -------------------------------- | ------------------------- | --------------------------- |
| `urn:ogc:def:parameter:OGC::101` | Latitude of rotated pole  | `grid_south_pole_latitude`  |
| `urn:ogc:def:parameter:OGC::102` | Longitude of rotated pole | `grid_south_pole_longitude` |
| `urn:ogc:def:parameter:OGC::103` | Axis rotation             | `grid_south_pole_angle`     |


### Formula
(Adapted from GRIB template 3.1):
The rotations are applied by first rotating the sphere through λ<sub>p</sub> about the geographic polar axis,
then rotating through (φ<sub>p</sub> − (−90°)) degrees so that the southern pole moved along the (previously rotated) Greenwich meridian,
and finally by rotating clockwise when looking from the southern to the northern rotated pole.

* Definitions:
  * (φ, λ) the (latitude, longitude) to rotate.
  * (φ<sub>p</sub>, λ<sub>p</sub>) the (latitude, longitude) of rotated pole.
  * θ the axis rotation about the new polar axis measured clockwise when looking from the southern to the northern pole.
* First rotation:
  * Δλ = λ − λ<sub>p</sub>
* To Cartesian coordinates
  * x = cos(φ) ⋅ cos(Δλ)
  * y = cos(φ) ⋅ sin(Δλ)
  * z = sin(φ)
* Useful trigonometric identities:
  * sin(φ<sub>p</sub> + 90°) =  cos(φ<sub>p</sub>)
  * cos(φ<sub>p</sub> + 90°) = −sin(φ<sub>p</sub>)
* Rotate φ<sub>p</sub> − (−90°)
  * x<sub>t</sub> =  cos(φ<sub>p</sub>) ⋅ z − sin(φ<sub>p</sub>) ⋅ x
  * y<sub>t</sub> =  y
  * z<sub>t</sub> = −cos(φ<sub>p</sub>) ⋅ x − sin(φ<sub>p</sub>) ⋅ z
* To spherical coordinates
  * R = √(x<sub>t</sub>² + y<sub>t</sub>²)
  *  φ<sub>t</sub> = atan2(z<sub>t</sub>, R)
  * Δλ<sub>t</sub> = atan2(y<sub>t</sub>, x<sub>t</sub>)
* Axis rotation
  * λ<sub>t</sub> = Δλ<sub>t</sub> − θ




## North pole rotation
Note: the PROJ parameters in this section will apply to the _inverse_ operation,
because of differences in the way those parameters are defined.
The `-I` option in PROJ definition below gets the forward (inverse of inverse) operation.

| Authority                 | Name or identifier                 |
| ------------------------- | ---------------------------------- |
| OGC identifier (proposal) | `urn:ogc:def:method:OGC::110`      |
| OGC name (proposal)       | `North pole rotation`              |
| WMO name                  | (none)                             |
| CF-convention name        | `rotated_latitude_longitude`       |
| PROJ definition           | `-I +proj=ob_tran +o_proj=longlat` |


### Parameters

| Identifier proposal              | Name proposal             | CF-convention name          | PROJ name  |
| -------------------------------- | ------------------------- | --------------------------- | ---------- |
| `urn:ogc:def:parameter:OGC::111` | Latitude of rotated pole  | `grid_north_pole_latitude`  | `+o_lat_p` |
| `urn:ogc:def:parameter:OGC::112` | Longitude of rotated pole | `grid_north_pole_longitude` | `+o_lon_p` |
| `urn:ogc:def:parameter:OGC::113` | Axis rotation             | `north_pole_grid_longitude` | `+lon_0`   |


### Formula
Similar to the south pole rotation except that the latitude rotation is (φ<sub>p</sub> − 90°) degrees
and the final rotation is clockwise when looking from the northern to the southern rotated pole.

* First rotation:
  * Δλ = λ − λ<sub>p</sub>
* To Cartesian coordinates
  * x = cos(φ) ⋅ cos(Δλ)
  * y = cos(φ) ⋅ sin(Δλ)
  * z = sin(φ)
* Useful trigonometric identities:
  * sin(φ<sub>p</sub> − 90°) = −cos(φ<sub>p</sub>)
  * cos(φ<sub>p</sub> − 90°) =  sin(φ<sub>p</sub>)
* Rotate φ<sub>p</sub> − 90°
  * x<sub>t</sub> = −cos(φ<sub>p</sub>) ⋅ z + sin(φ<sub>p</sub>) ⋅ x
  * y<sub>t</sub> =  y
  * z<sub>t</sub> =  cos(φ<sub>p</sub>) ⋅ x + sin(φ<sub>p</sub>) ⋅ z
* To spherical coordinates
  * R = √(x<sub>t</sub>² + y<sub>t</sub>²)
  *  φ<sub>t</sub> = atan2(z<sub>t</sub>, R)
  * Δλ<sub>t</sub> = atan2(y<sub>t</sub>, x<sub>t</sub>)
* Axis rotation
  * λ<sub>t</sub> = Δλ<sub>t</sub> + θ


#### Open question
Axis rotation has been defined as "clockwise when looking from the northern to the southern rotated pole"
for symmetry with the definition in South pole case.
But it causes a change of sign (+θ instead of −θ) for the last term in above formula.
Should be change the definition in a way that keep the same sign,
for example by replacing "clockwise" by "counter-clockwise" in the North pole case?

#### Open issue
`ucar.unidata.geoloc.projection.RotatedPole` in UCAR netCDF library version 5.5.2 gives results
with an offset of 180° in longitude values compared to what we would expect from a geometrical reasoning:
if we rotate the pole to 60°N, then latitude of 59°N on Greenwich meridian become only 1° below new pole,
i.e. 89°N, but still on the same meridian (Greenwich) because we did not cross the pole. Conversely 61°N
is still at 89°N relative to the rotated pole, but on the other side of the pole, i.e. at longitude 180°.
But `RotatedPole` gives opposite longitude values (180° and 0° respectively).

The "South pole rotation" method of netCDF library is consistent with this geometrical reasoning.
We do not know if the difference observed for "North pole rotation" is intended or not.


## Rotated ellipsoidal coordinate system
The coordinate operation results are (latitude, longitude) coordinates on a rotated sphere.
The "Geodetic latitude" and "Geodetic longitude" axis names of coordinate system EPSG:6422
may not be applicable anymore.
The following defines a new coordinate system with different axis names.

| Object class           | Identifier proposal         | Name proposal           |
| ---------------------- | --------------------------- | ----------------------- |
| `EllipsoidalCS`        | `urn:ogc:def:cs:OGC::100`   | Ellipsoidal rotated CS. |
| `CoordinateSystemAxis` | `urn:ogc:def:axis:OGC::101` | Rotated latitude        |
| `CoordinateSystemAxis` | `urn:ogc:def:axis:OGC::102` | Rotated longitude       |

### Open issue
Axis names are constrained by [ISO 19111 table 27](http://docs.opengeospatial.org/as/18-005r5/18-005r5.html#table_27),
and above names are not in the list of permitted axis names.



# Example with COSMO meteorological model
The [RotatedPole.xml](RotatedPole.xml) file gives an example of CRS definition in GML.
It uses the following identifiers in COSMO namespace.
In this example, those definitions are considered specific to COSMO model
and would not be stored on an OGC or WMO server.
Instead the COSMO project would need to maintain their own registry.

| Object class    | Identifier example                           | Name example                            |
| --------------- | -------------------------------------------- | --------------------------------------- |
| `DerivedCRS`    | `urn:ogc:def:crs:COSMO::101`                 | COSMO-DWD rotated pole grid             |
| `Conversion`    | `urn:ogc:def:coordinateOperation:COSMO::101` | COSMO-DE pole rotation                  |
| `GeodeticCRS`   | `urn:ogc:def:crs:COSMO::100`                 | COSMO DWD base geodetic CRS (spherical) |
| `GeodeticDatum` | `urn:ogc:def:datum:COSMO::100`               | COSMO DWD geodetic datum (spherical)    |
| `Ellipsoid`     | `urn:ogc:def:ellipsoid:COSMO::100`           | COSMO DWD models sphere                 |


The GML file passes XML validation. Parsing with Apache SIS gives the following WKT:

```
GEODCRS["COSMO-DWD rotated pole grid",
  BASEGEODCRS["COSMO DWD base geodetic CRS (Spherical)",
    DATUM["COSMO DWD geodetic datum (Spherical)",
      ELLIPSOID["DWD Models Sphere", 6371229.0, 0.0, LENGTHUNIT["metre", 1]]],
      PRIMEM["Greenwich", 0.0, ANGLEUNIT["degree", 0.017453292519943295]]],
  DERIVINGCONVERSION["COSMO-DE pole rotation",
    METHOD["North pole rotation", ID["OGC", 110]],
    PARAMETER["Latitude of rotated pole", 40.0, ANGLEUNIT["degree", 0.017453292519943295]],
    PARAMETER["Longitude of rotated pole", -170.0, ANGLEUNIT["degree", 0.017453292519943295]],
    PARAMETER["Axis rotation", 0.0, ANGLEUNIT["degree", 0.017453292519943295]]],
  CS[ellipsoidal, 2],
    AXIS["Rotated latitude (B)", north, ORDER[1]],
    AXIS["Rotated longitude (L)", east, ORDER[2]],
    ANGLEUNIT["degree", 0.017453292519943295],
  SCOPE["Atmospheric model data."],
  AREA["Central Germany."],
  BBOX[47.00, 5.00, 55.00, 15.00],
  ID["COSMO", 101],
  REMARK["Used with grid spacing of 0.025° in rotated coordinates."]]
```

Converting a central point in Germany (51.1657°N 10.4515°E) gives about (1.1666°N 179.71682°W).
It was tested with both Apache SIS using XML definition and PROJ using following command line:

```
cs2cs -I "EPSG:4326" +to +type=crs +proj=ob_tran +o_proj=longlat +datum=WGS84 +no_defs +o_lat_p=40 +o_lon_p=-170
```
