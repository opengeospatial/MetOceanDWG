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
This is the operation defined by GRIB template 3.1.
Also used as the base operation from which the "North pole rotation" case will be derived.

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
The rotations are applied by first rotating the sphere through λ<sub>p</sub> about the geographic polar axis,
then rotating through (φ<sub>p</sub> − (−90°)) degrees so that the southern pole moved along the (previously rotated) Greenwich meridian,
and finally by rotating clockwise when looking from the southern to the northern rotated pole.
The 180° rotated meridian runs through both the geographical and the rotated South pole.

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
  *  φ<sub>t</sub> = asin(z<sub>t</sub>)
  * Δλ<sub>t</sub> = atan2(y<sub>t</sub>, x<sub>t</sub>)
* Axis rotation
  * λ<sub>t</sub> = Δλ<sub>t</sub> − θ


### Inverse operation
Conversion from the rotated CRS to the geographic CRS can be done using the same "South pole rotation" method,
but with all parameters with subscript <var>p</var> replaced by following parameters with subscript <var>g</var>:

* φ<sub>g</sub>  = φ<sub>p</sub>
* λ<sub>g</sub>  = copySign(180, θ<sub>p</sub>) - θ<sub>p</sub>
* θ<sub>g</sub>  = copySign(180, λ<sub>p</sub>) - λ<sub>p</sub>

`copySign(180, x)` is a function returning 180 with the same sign than <var>x</var>.
In above formulas, it is used for applying a shift of 180° (for antipodal longitude)
in such a way that the resulting angle stay in the [−180 … 180]° range.



## North pole rotation
We found no authoritative source defining this operation.
This section describes what seems a common usage.

| Authority                 | Name or identifier                 |
| ------------------------- | ---------------------------------- |
| OGC identifier (proposal) | `urn:ogc:def:method:OGC::110`      |
| OGC name (proposal)       | `North pole rotation`              |
| WMO name                  | (none)                             |
| CF-convention name        | `rotated_latitude_longitude`       |
| PROJ definition           | `-I +proj=ob_tran +o_proj=longlat` |

Note: the PROJ parameters in this section will apply to the _inverse_ operation,
because proposed OGC parameters are defined as latitude/longitude of the rotated pole,
while PROJ parameters are latitude/longitude of the north pole expressed in the rotated CRS.
The `-I` option in above PROJ definition gets the forward (inverse of inverse) operation.


### Parameters

| Identifier proposal              | Name proposal             | CF-convention name          | PROJ name          |
| -------------------------------- | ------------------------- | --------------------------- | ------------------ |
| `urn:ogc:def:parameter:OGC::111` | Latitude of rotated pole  | `grid_north_pole_latitude`  | `+o_lat_p`         |
| `urn:ogc:def:parameter:OGC::112` | Longitude of rotated pole | `grid_north_pole_longitude` | `+o_lon_p`         |
| `urn:ogc:def:parameter:OGC::113` | Axis rotation             | `north_pole_grid_longitude` | `+lon_0` antipodal |

**Note on PROJ parameters:**
The value given to PROJ `+lon_0` parameter needs to be the axis rotation value with a shift of ±180°.
If there is no axis rotation, then `+lon_0=180` needs to be specified.
Alternatively, the `-I` option can be avoided by swapping the `+lon_0` and `+o_lon_p` values
(the +180 offset stay on `+lon_0`).


### Formula
The rotations are applied by first rotating the sphere through λ<sub>p</sub> about the geographic polar axis,
then rotating through (φ<sub>p</sub> − 90°) degrees so that the northern pole moved along the (previously rotated) Greenwich meridian,
and finally by rotating clockwise when looking from the northern to the southern rotated pole.
The 0° rotated meridian is defined as the meridian that runs through both the geographical and the rotated North pole.

No formula is derived for the North pole case.
Instead the "South pole rotation" method should be used for rotating the antipodal point of the rotated north pole.
This approach automatically inserts a shift of 180° in longitude values, as required for above 0° meridian definition.
More specifically, a "North pole rotation" with parameters identified by the subscript <var>N</var> below
can be implemented by a "South pole rotation" with the following parameters:

* φ<sub>p</sub>  = −φ<sub>N</sub>
* λ<sub>p</sub>  =  λ<sub>N</sub> − copySign(180, λ<sub>N</sub>)
* θ<sub>p</sub>  = −θ<sub>N</sub>


### Inverse operation
Conversion from the rotated CRS to the geographic CRS can be done using the same "North pole rotation" method,
but with all parameters with subscript <var>N</var> replaced by following parameters with subscript <var>g</var>:

* φ<sub>g</sub>  = φ<sub>N</sub>
* λ<sub>g</sub>  = θ<sub>N</sub>
* θ<sub>g</sub>  = λ<sub>N</sub>


#### Open question
Axis rotation has been defined as "clockwise when looking from the northern to the southern rotated pole"
for symmetry with the definition in South pole case.
It also produces more symetrical formulas in this section.
But we found no authoritative source saying if the rotation should be clockwise or counter-clockwise.


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


### XML definition files
Two components could be subject to OGC standardisation and are provided as GML files:

* [PoleRotation.xml](PoleRotation.xml) defines the operation method described in this page.
* [RotatedCS.xml](RotatedCS.xml) defines the ellipsoidal coordinate system described above.

Above are only components, not fully defined CRS.
While above components could be used for any CRS based on rotated pole,
a fully CRS can only be specific to a particular meteorological model (or other application).
The next section gives an example.


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

Converting a central point in Germany (51.1657°N 10.4515°E) gives about (1.1666°N 0.2832°W).
It can be tested with UCAR, PROJ and Apache SIS as below:


## UCAR netCDF library
The UCAR library has been used as a reference for both PROJ `ob_tran` mapping and for Apache SIS.
The code below is a snippet of Java code for converting the above-cited test coordinates.
Note that this code provides no datum information; it is purely the pole rotation method with nothing else.

```java
var point    = ucar.unidata.geoloc.LatLonPoint.create(51.1657, 10.4515);
var rotation = new ucar.unidata.geoloc.projection.RotatedPole(40, -170);
System.out.println(rotation.latLonToProj(point));
```

Output:

```
.2831 1.166
```


## PROJ command line
The following command-line can be executed on a Unix system.
Note that the COSMO model uses a sphere which is close, but not identical,
to _International 1924 Authalic Sphere_ (`urn:ogc:def:datum:EPSG::6053`).
We use EPSG:4053 CRS as a close approximation.
However the `ob_tran` method ignores the ellipsoid, because formulas are applied on a sphere
(like all other implementations tested on this page)
and the source and target coordinates are geographic coordinates on the same sphere.
Replacing "EPSG:4053" (International 1924) by "EPSG:4326" (WGS 1984) on PROJ 8.2.0
does not change the output coordinates.

```shell
cs2cs -I "EPSG:4053" +to +type=crs +proj=ob_tran +o_proj=longlat +R=6371229 +no_defs \
      +o_lat_p=40 +o_lon_p=-170 +lon_0=180 -f %g <<< "10.4515 51.1657"
```

Output:

```
1.16655	0.283179 0
```

**Reminder:** input coordinates (or output coordinates in inverse operation)
above are **not** WGS 1984 (EPSG:4326) geographic coordinates.
They are closer to "International 1924" coordinates.
There is a potential difference of about 20 km in Central Germany.
If desired, transformation from/to WGS 84 can be added using more PROJ parameters.
See Apache SIS case below for more discussion on the impact of the datum on output coordinates.


## Apache SIS
Apache SIS can read GML, so it can be used for testing the [RotatedPole.xml](RotatedPole.xml) definition file directly.
In the following command-line, replace `RotatedPole.xml` by
[this URL](https://raw.githubusercontent.com/opengeospatial/MetOceanDWG/main/MetOceanDWG%20Projects/Authority%20Codes%20for%20CRS/RotatedPole.xml)
(edit URL path if the location of `RotatedPole.xml` file changed) or a local copy of that file.
This command requires SIS version 1.2 or above,
because the "North pole rotation" method was not available in SIS 1.1.

```shell
sis transform --sourceCRS EPSG:4053 --targetCRS "RotatedPole.xml" <<< "51.1657, 10.4515"
```

Output:

```
# Source:      Unspecified datum based upon the International 1924 Authalic Sphere (EPSG:4053)
# Destination: COSMO-DWD rotated pole grid (COSMO:101)
# Operations:  Abridged Molodensky → COSMO-DE pole rotation (COSMO:101)
# Accuracy:    3000.0 metres

Rotated latitude (°), Rotated longitude (°)
    1.166554714,      0.283179132
```

### Discussion about datum
Above command used "International 1924 Authalic Sphere", which is a sphere of radius 6371228 meters.
This is very close to the COSMO model, which uses a sphere of radius 6371229 meters, but not identical.
The one meter difference is the reason why Apache SIS inserted an "Abridged Molodensky" operation before the pole rotation.
Since that operation is not referenced in the EPSG database, this is considered as a datum shift of unknown accuracy.
In such case SIS conservatively reports a 3 km accuracy as the worst case scenario.
The result is nevertheless close to UCAR and PROJ results because of similarity between the two spheres.

If we replace EPSG:4053 (International 1924) by EPSG:4326 (WGS 84) in the command-line:

```shell
sis transform --sourceCRS EPSG:4326 --targetCRS "RotatedPole.xml" <<< "51.1657, 10.4515"
```

Then the result become:

```
# Source:      WGS 84 (EPSG:4326)
# Destination: COSMO-DWD rotated pole grid (COSMO:101)
# Operations:  Abridged Molodensky → COSMO-DE pole rotation (COSMO:101)
# Accuracy:    3000.0 metres

Rotated latitude (°), Rotated longitude (°)
    0.978570118,      0.284314319
```

The difference between the two results is about 20 km.
Users should be careful about whether their tools apply a datum shift or not.
In this example, the datum shift is only a change of ellipsoid axis lengths and flattening factors,
without Bursa-Wolf parameters or Helmert transformation.
