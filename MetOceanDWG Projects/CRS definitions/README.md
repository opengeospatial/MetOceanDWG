# CRS definitions proposal
It would be desirable to register the following temporal CRSs with the OGC Naming Authority,
in the following priority order:

* Indexed Time, such as T+hh, where _T_ is the epoch/datum and _hh_ is the Unit of Measurement.
  There may be a need for further metadata, corresponding to pixel as point centre, or corner.
  * Note: the TSML standard has a table indicating whether a time instant represents earliest,
    midpoint or latest time of an interval. DGGS have a use case for this TCRS.
* Countdown Time, such as T-10, 9, 8, …2, 1, ZERO, T+1, T+2, _etc._
  Any pause/hold of the countdown is a discontinuity in the datum/epoch shift
  (i.e., in the coordinate transformation from countdown to UNIX time for instance).
  The Unit of Measurement may be other than Seconds, such as Days, e.g. 6 June 1944, D-dd.
  This later case had a discontinuous datum shift as the original plan was for 5 June,
  but delayed by the weather.
* A 360 day TCRS. This is probably better as a calendar rather than a CRS.
  If there was only one Unit of Measurement (UoM), it would be either a Julian day or perhaps Julian year,
  but years with 360 day subdivisions seems to be a calendar with 12 months of 30 days as a third UoM.

## Starting point
The following files are copies of files of the same from OGC definition server
at [www.opengis.net/def/crs/OGC/0/](https://www.opengis.net/def/crs/OGC/0/),
except for the `.xml` suffix added to filenames.
Then, the files have been modified as seen from Git history and described in the next section.

* [def/crs/OGC/0/JulianDate](https://www.opengis.net/def/crs/OGC/0/JulianDate)
* [def/crs/OGC/0/TruncatedJulianDate](https://www.opengis.net/def/crs/OGC/0/TruncatedJulianDate)
* [def/crs/OGC/0/UnixTime](https://www.opengis.net/def/crs/OGC/0/UnixTime)
* [def/crs/OGC/0/Index1D](https://www.opengis.net/def/crs/OGC/0/Index1D)
* [def/crs/OGC/0/Index2D](https://www.opengis.net/def/crs/OGC/0/Index2D)
* [def/crs/OGC/0/Index3D](https://www.opengis.net/def/crs/OGC/0/Index3D)
* [def/crs/OGC/0/Index4D](https://www.opengis.net/def/crs/OGC/0/Index4D)

## Modifications
The most important changes in above files are:

* [Fix erroneous `pixelInCell` value](https://github.com/Geomatys/MetOceanDWG/commit/481f41e0a5672325131e50b9dd062753c237611e):
  the `<ImageDatum>` description said _"(…) with the origin at the lower bound of the origin cell in the axis"_,
  but the code list value was `cellCenter` instead of `cellCorner`.
* [Replace patterns](https://github.com/Geomatys/MetOceanDWG/commit/57ee87e42f7e12be3ade98b47be81b8926ed3a3d)
  such as `%indexnd-td-corner%` by identifiers as URLs such as `http://www.opengis.net/def/datum/OGC/0.2/CellCorner`.
* [Replace the `indexedAxisPositive` axis direction](https://github.com/Geomatys/MetOceanDWG/commit/cf5e2a0d6409c3704ffa11e3ecafb6551d965221)
  by the directions specified by ISO 19111 such as `columnPositive`, `rowPositive`, `up` and `future`.
  The purpose of this change is to give a different direction to each axis of a coordinate system.
  Before this change, all axes of all indexed coordinate system had the same custom direction,
  which can cause some software to consider that all axes are colinear.
* The above change introduces a stronger relationship between grid axes and real-world axes for dimensions above 2
  (the two first dimensions stay more abstract as column index and row index). Consequently,
  [rename `Index3D` as `SpatialIndex3D` and `Index4D` as `SpatioTemporalIndex3D`](https://github.com/Geomatys/MetOceanDWG/commit/beb332d1b597a6765e122d52476190106dd87acd).
  We will also need a `SpatioTemporalIndex3D` and maybe more variants, but this is left for a future commit.
* As the two first dimensions stay more abstract, and as grid axes are generally related to real world axes by an affine transform,
  [replace `CartesianCS` by `AffineCS`](https://github.com/Geomatys/MetOceanDWG/commit/61ee2211ead02989dc1325633ff7f890385f0677).
  It does not block the use of Cartesian coordinate systems as they are special cases of affine coordinate systems.

More minor changes are:

* Upgraded the version number from 0.1 to 0.2
  [in remarks](https://github.com/Geomatys/MetOceanDWG/commit/e8596ece38251eec42df34b3cb5655c1a9826fd7)
  and [in URLs](https://github.com/Geomatys/MetOceanDWG/commit/70bf019f8c6cd24be55711bea529400f6f9b217f).
* [Replaced "not known" scopes](https://github.com/Geomatys/MetOceanDWG/commit/4abe148e6b9ee1d5ff962a6ee60c0fc6fd9d2ec3)
  by texts such as _"For referencing cells in a two-dimensional grid"_.
