## SRTM-GeoTIFF
A simple and versatile Python snippet to read elevation data from raster-based SRTM files from data sources such as:

`USGS EarthExplorer` `Japan Space Systems` `NASA ASTER GDEM`  `ALOS AW3D30` `OpenTopography`

File types supported by this snippet: Public Domain- `GeoTIFF`, Military- `DTED`, ESRI- `BIL` and legacy- `HGT`

Data type supported by this snippet: raster files with few restrictions on pixel dimensions or aspect ratios. It can handle file types like: NASA/JSS ASTER GDEM 3601x3601. ALOS AW3D30 3600x3600. USGS 3601x3601, 1801x3601, 1201x1201 and 601x1201.

This snippet makes use of GDAL's [GetGeoTransform](https://gdal.org/tutorials/geotransforms_tut.html) (Affine Transformation) to translate the given latitude, longitude into pixel indices. It also handles LZW compressed rasters automagically.

The more challenging task is perhaps to find out which tile/filename to use for a particular lat/lon location, especially when each data source uses their own file naming convention. For personal hobby use, I use `GeoTIFF` format because the [file naming convention](/library/tilename.py) (with small additional regex manipulations) is similar to the original SRTM .hgt files. For ALOS's 3600x3600 non-overlapping tiles, a modified algorithm [tile_alos.py](/library/tile_alos.py) is necessary to parse given latitude and longitude to select the correct filename.

Sources of ASTER GDEM (Global Digital Elevation Model) in GeoTIFF format are:

[USGS EarthExplorer](https://earthexplorer.usgs.gov/). Here is a [primer](/EarthExplorer.md) on how to download GeoTIFF from USGS EarthExplorer.

[Japan Space Systems ASTER GDEM](https://gdemdl.aster.jspacesystems.or.jp/index_en.html).

[NASA's Earth Data](https://search.earthdata.nasa.gov/search/) 

### Example:
```
Using Python 3.10.6 and GDAL 3.4.3; Windows Subsystem Linux (Ubuntu 22.04)
>>>import srtm1
>>>srtm1.read( './N46E008.tif', 46.854539, 8.49701)
2691 (Lucern, Switzerland)
>>>srtm1.read( './N49W123.tif', 49.68437, -122.14162 )
618  (British Columbia, Canada)
>>>srtm1.read( './S33W071.tif', -32.653197, -70.0112 )
6930 (Aconcagua, Argentina)
>>>srtm1.read( './S36E149.tif', -35.2745, 149.09752 )
801  (Canberra, Australia)
```
### Determine which tile to use:

For NASA and USGS (1-pixel overlapping tiles):
```
>>> import tilename
>>> tilename.find( 49.6, -122.1 )
>>> 'N49W123.tif'
>>> tilename.find( 49.0, -122.1 )
>>> 'N49W123.tif'
```
For ALOS (non-overlapping tiles):
In words, if the latitude is exactly an integer, then use 1 tile further south compared to NASA's/USGS's counterpart.
```
>>> import tile_alos
>>> tile_alos.find ( 49.6, -122.1 )
>>> 'N49W123.tif'
>>> tile_alos.find ( 49.0, -122.1 )
>>> 'N48W123.tif'
```
Subtle difference, but critical to avoid index out of range errors when latitude is an integer.
