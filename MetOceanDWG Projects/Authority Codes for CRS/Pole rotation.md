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

* **Proposal:** `South pole rotation`
* **WMO name:** `Rotated Latitude/longitude`
* **CF name:**  (none) — UCAR netCDF library uses `rotated_latlon_grib`


### Parameters

| Proposal                  | Name in UCAR library        |
| ------------------------- | --------------------------- |
| Latitude of rotated pole  | `grid_south_pole_latitude`  |
| Longitude of rotated pole | `grid_south_pole_longitude` |
| Axis rotation             | `grid_south_pole_angle`     |


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
  * sin(φ + 90°) =  cos(φ)
  * cos(φ + 90°) = −sin(φ)
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

* **Proposal:** `North pole rotation`
* **WMO name:**  (none)
* **CF name:**  `rotated_latitude_longitude` (see note below)
* **PROJ:**     `-I +proj=ob_tran +o_proj=longlat`


### Parameters

The PROJ parameters define the _inverse_ operation,
because of differences in the way those parameters are defined.
The `-I` option in above PROJ definition gets the forward (inverse of inverse) operation.

| Proposal                  | CF-convention name          | PROJ name  |
| ------------------------- | --------------------------- | ---------- |
| Latitude of rotated pole  | `grid_north_pole_latitude`  | `+o_lat_p` |
| Longitude of rotated pole | `grid_north_pole_longitude` | `+o_lon_p` |
| Axis rotation             | `north_pole_grid_longitude` | `+lon_0`   |


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
  * sin(φ − 90°) = −cos(φ)
  * cos(φ − 90°) =  sin(φ)
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
