# CRS definitions proposal
It would be desirable to register the following temporal CRSs with the OGC Naming Authority,
in the following priority order:

* Indexed Time, such as T+hh, where _T_ is the epoch/datum and _hh_ is the Unit of Measurement.
  There may be a need for further metadata, corresponding to pixel as point centre, or corner.
  * Note: the TSML standard has a table indicating whether a time instant represents earliest,
    midpoint or latest time of an interval. DGGS have a use case for this TCRS.
* Countdown Time, such as T-10, 9, 8, …2, 1, ZERO, T+1, T+2, _etc._
  Any pause/hold of the countdown is a discontinuity in the datum/epoch shift
  (i.e., in the coordinate transformation from coutdown to UNIX time for instance).
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
Then, the files have been modified as seen from Git history.

* [def/crs/OGC/0/JulianDate](https://www.opengis.net/def/crs/OGC/0/JulianDate)
* [def/crs/OGC/0/TruncatedJulianDate](https://www.opengis.net/def/crs/OGC/0/TruncatedJulianDate)
* [def/crs/OGC/0/UnixTime](https://www.opengis.net/def/crs/OGC/0/UnixTime)
* [def/crs/OGC/0/Index1D](https://www.opengis.net/def/crs/OGC/0/Index1D)
* [def/crs/OGC/0/Index2D](https://www.opengis.net/def/crs/OGC/0/Index2D)
* [def/crs/OGC/0/Index3D](https://www.opengis.net/def/crs/OGC/0/Index3D)
* [def/crs/OGC/0/Index4D](https://www.opengis.net/def/crs/OGC/0/Index4D)
