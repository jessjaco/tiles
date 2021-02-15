# Making slippy tiles

First, note that tiles are either raster or vector based. Raster tiles are more common

## Raster Tiles
There are two basic steps to create tiled raster layers. First, symbolize the raster as an RGB image. Then export to tiles.

1. via Google Earth Engine

Images can be symbolized using `visParams` or a styled-layer descriptor in xml format.

Export a tiled image asset using `Export.map`. E.g.

```
var img = ee.Image(<path>);
var map = img.sldStyle(<sld>);
Export.map.toCloudStorage(
)
```

The output can be served directly from the cloud bucket if the tiles are made public (please note that GEE will not export to a public repo!).

2. Using OGR/GDAL

Rasters can be converted to RGB images using . Then, you can export to xyz tiles using `gdal_translate.py`. For instance:


3. Using R

I'm not sure this is a common workflow, but R is good at making color ramps, 

4. Other methods

Upload to mapbox. QGIS, ArcGIS?? Not sure...


## Mapbox Vector Tiles

Vector tiles offer advantages over raster tiles as they can store multiple pieces of information (and in numeric or string format) for each feature. They also can be symbolized dynamically in a data-driven way. Mapbox-gl is the primary platform supporting vector tiles, though there is some support in leafet (and others??). 

1. Upload to Mapbox
You can upload geojson (and shapefiles?) and mapbox will turn them into vector tiles which they also host. I don't use this pathway much though because there are limits on feature and file size and historically there was cost to serve tiles from mapbox (I think this changed in mapbox-gl v1).

2. Use tippecanoe
Tippecanoe is a tool to create vector tiles from geojson files. Input data _must_ be in EPSG:4326. Tippecanoe is powerful and dynamic and I use it for most tile export. 

3. Use gdal?
