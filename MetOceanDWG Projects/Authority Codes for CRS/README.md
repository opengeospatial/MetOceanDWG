# Authority Codes for CRS definitions

This directory proposes a mapping from standard definitions provided by organizations
such as the World Meteorological Organization (WMO) to Coordinate Reference System objects.
The use of alternative authorities aims to complement the use of EPSG codes
when CRS commonly used in the meteorological domain is not found in the EPSG geodetic dataset.
Following discussion takes _Rotated pole_ as an example but can apply to any projection method
currently not in the EPSG database.

## Problem statement
[ISO 19162 §14.3.2](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html#116)
provides an example of a geographic CRS with pole rotation.
Following WKT is derived from that example and discussion on the mailing list.
The important things for this discussion are the `METHOD` and `PARAMETER` elements.

```
GEOGCRS["Atlantic Pole",
  BASEGEOGCRS["Based upon sphere",
    DATUM["Based on GRS 1980 Authalic Sphere",
    ELLIPSOID["GRS 1980 Authalic Sphere", 6371007, 0, LENGTHUNIT["metre", 1]]],
  DERIVINGCONVERSION["Atlantic pole",
    METHOD["Pole rotation", ID["Authority", 1234]],
    PARAMETER["Latitude of rotated pole", 52.0,
      ANGLEUNIT["degree", 0.0174532925199433]],
    PARAMETER["Longitude of rotated pole", -30.0,
      ANGLEUNIT["degree", 0.0174532925199433]],
    PARAMETER["Axis rotation", -25.0,
      ANGLEUNIT["degree", 0.0174532925199433]]],
  CS[ellipsoidal, 2],
    AXIS["latitude", north, ORDER[1]],
    AXIS["longitude", east, ORDER[2]],
    ANGLEUNIT["degree", 0.0174532925199433]]
```

A CRS definition, no matter if expressed in GML or WKT,
have 3 components that are controlled vocabulary defined by external authorities
(i.e. not code lists or enumerations defined by ISO 19111 or WKT specification):

* The operation method.
* The operation parameters.
* The datum (in some cases).

The operation method (“Pole rotation” in above example) identifies the set of equations.
It would be too complex and too verbose to encode the equations in a WKT or GML definition.
For this reason, the `METHOD` element is only a reference to formulas published by an authority.
Two sources of formulas are:

* IOGP Publication 373-7-2 – Geomatics Guidance Note number 7, part 2:
  _Coordinate Conversions and Transformations including Formulas._
  https://epsg.org/guidance-notes.html
* OGC 01-009: OpenGIS Implementation Specification: _Coordinate Transformation Services,_
  section 10.6.1: Cartographic Projection Transform Parameters.

The OGC 01-009 document is legacy (it actually does not give all formulas) and should be replaced by IOGP document.
But the operation and parameter names given by OGC 01-009 §10.6.1 are still in common use.

**Example:** when a WKT contains the following element:

```
METHOD[“Lambert Conic Conformal (1SP)”, ID[“EPSG”, 9801]]
```

It actually refers to formulas published in section 3.1.1.2 of IOGP 373-7-2 (March 2020).
The parameters associated to this operation method are listed in table 2 of that document,
and are also given in the EPSG geodetic database.
Those formulas have to be hard-coded in the map projection library;
it would be too difficult to design a WKT, GML or JSON format that allows the encoding of complete formulas.

Using `METHOD` and `PARAMETER` elements,
it is possible to write inter-operable definitions for CRS that are not in the EPSG database,
for example by providing parameter values that are different than the values of all CRS in the EPSG database.
But the `METHOD` (together with its set of parameters) must be defined by some authority,
otherwise the CRS definition would be specific to a particular implementation.

OGC 05-010 _URNs of definitions in ogc namespace_ recommendation paper defines object types
for operations methods and their parameters.
Examples (note that 9801, 8801 and 8802 are not CRS codes in this context):

| Example                            | Meaning                       |
| ---------------------------------- | ----------------------------- |
| `urn:ogc:def:method:EPSG::9801`    | Lambert Conic Conformal (1SP) |
| `urn:ogc:def:parameter:EPSG::8801` | Latitude of natural origin    |
| `urn:ogc:def:parameter:EPSG::8802` | Longitude of natural origin   |

The problem is that there is currently no authority code for “Pole Rotation” operation method.
It can been seen from the `ID[“Authority”, 1234]` placeholder in the WKT example published in ISO 19162.


## Proposal

Define URN in “urn:ogc:def:method:WMO” namespace as references to templates published in the WML Manual of Codes.
This proposal is detailled in the following pages:

* [WMO as a source of authority codes for coordinate operations](WMO.md)

Following page is a more exploratory work:

* [Pressure as Z coordinate](Pressure%20as%20Z%20coordinate.md)
