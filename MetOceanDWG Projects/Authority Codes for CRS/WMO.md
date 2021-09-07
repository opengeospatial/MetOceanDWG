# WMO as a source of authority codes for coordinate operations

The problem statement is described in the [README](README.md) page.
This page describes the proposal using WMO as a source of authoritative definitions.



## Proposal for coordinate operations

Define URN in “urn:ogc:def:method:WMO” namespace as below as references to templates in the following document:

* [WMO. Manual on Codes. No. 306, volume 1.2, parts B and C.](https://library.wmo.int/doc_num.php?explnum_id=10310)

Syntax (edition is optional; if omitted, latest edition is assumed):

```
urn:ogc:def:method:WMO:edition:template
```

Example for Pole Rotation:

```
urn:ogc:def:method:WMO:2019:3.1
```

For making the definition clearer, OGC should publish the operation method name together with its parameters.
The following is adapted from WMO template 3.1:

> Operation method name: “rotated latitude/longitude”
> Operation method code: urn:ogc:def:method:WMO:2019:3.1
> Parameters:
>
> * Latitude of the southern pole of projection
> * Longitude of the southern pole of projection
> * Angle of rotation of projection
>
> Three parameters define a general latitude/longitude coordinate system,
> formed by a general rotation of the sphere. One choice for these parameters is:
>
> (a) The geographic latitude in degrees of the southern pole of the coordinate system, θ p ;
> (b) The geographic longitude in degrees of the southern pole of the coordinate system, λ p ;
> (c) The angle of rotation in degrees about the new polar axis (measured clockwise when
> looking from the southern to the northern pole) of the coordinate system, assuming the new
> axis to have been obtained by first rotating the sphere through λ p degrees about the geographic
> polar axis, and then rotating through (90 + θ p ) degrees so that the southern pole moved along
> the (previously rotated) Greenwich meridian.

Templates 3.2 and 3.3 may also be provided.



### Open questions:

 * What kind of OGC document would it be?
 * How would it appears in OGC Naming Authority?
 * Do we have a WMO contact for checking with them if this proposal makes sense?
