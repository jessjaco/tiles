# Making slippy tiles

First, note that tiles are either raster or vector based. Raster tiles are more common

## Raster Tiles
There are two basic steps to create tiled raster layers. First, symbolize the raster as an RGB image. Then export to tiles.

1. via Google Earth Engine

Images can be symbolized using `visParams` or a styled-layer descriptor in xml format.

Export a tiled image asset using `Export.map`. E.g.

```
ramp = < some sld >
var img = ee.Image(ee_path).sldStyle(ramp)

# OR

var img = ee.Image(ee_path).visualize({palette: ['#ffffcc','#003b1e']});

# Then

Export.map.toCloudStorage(
  image=img,
  bucket='my-bucket',
  path='tiles/my_raster',
  writePublicTiles=True,
  maxZoom=12)
```
I usually don't go above 12 max zoom. If testing, you can stick with 10 (or under!). Here I use an sld as they tend to be better for specific values, but are definitely harder to write. 

The output can be served directly from the cloud bucket if the tiles are made public (please note that GEE will not export to a public bucket!). Otherwise, copy them from GCS to wherever they should be served from. 

2. Using R & GDAL
R is good for making color ramps and creating RGB tiffs.

First, you need to get a list of colors and breaks. You can do this manually, you can symbolize it in QGIS, ArcGIS, GEE, etc and just write the values down, or you can use something like the following. Here I use 10 quantile breaks and a ramp going from light yellow (#ffffcc) to dark green (#003b1e). I'd just note that this can take a while for large rasters, so you can subsample say 1e6 values from `values(some_raster)` if needed.
```
breaks <- seq(0,1,length.out=10)
break_vals <- quantile(values(some_raster), prob=breaks, na.rm=TRUE)
colors <- c('#ffffcc', '#003b1e')
ramp <- colorRampPalette(colors)(length(break_vals))
```
Now `break_vals` and `ramp` are vectors of values and tiles respectively:
```
> break_vals
 [1] 0.00000000 0.05263158 0.10526316 0.15789474 0.21052632 0.26315789
 [7] 0.31578947 0.36842105 0.42105263 0.47368421 0.52631579 0.57894737
[13] 0.63157895 0.68421053 0.73684211 0.78947368 0.84210526 0.89473684
[19] 0.94736842 1.00000000

> ramp
 [1] "#FFFFCC" "#F1F4C2" "#E4EAB9" "#D6E0B0" "#C9D5A7" "#BBCB9E" "#AEC195"
 [8] "#A1B68B" "#93AC82" "#86A279" "#789770" "#6B8D67" "#5D835E" "#507854"
[15] "#436E4B" "#356442" "#285939" "#1A4F30" "#0D4527" "#003B1E"
```

Then use these to create an RGB image:
```
library(raster)
rgb_raster <- RGB(some_raster, col=ramp, breaks=break_vals, colNA='black')
options = c("COMPRESS=LZW", "BIGTIFF=YES", "TILED=TRUE", "BLOCKXSIZE=256", "BLOCKYSIZE=256")
writeRaster(rgb_raster,
    filename='raster_RGB.tif', 
    datatype='INT1U',
    NAflag=0,
    overwrite=TRUE,
    options=options)
```
Some of the above options are particular to a workflow I had, but they should work.

Once you have the rgb raster, you can use the `gdal2tiles.py` utility to export to tiles. This is part of OGR/GDAL, so you need to install that. Then you can create tiles with something like

`gdal2tiles.py -z 1-11 raster_RGB.tif`

The -z argument is the zoom level. The meters / pixel of each zoom level is described [here](https://wiki.openstreetmap.org/wiki/Zoom_levels). Usually I don't go above 10, 11 or 12. You don't typically need extremely high resolution for a web map.

4. Other methods

Upload to mapbox. QGIS, ArcGIS. To be continued...


## Mapbox Vector Tiles

Vector tiles offer advantages over raster tiles as they can store multiple pieces of information (and in numeric or string format) for each feature. They also can be symbolized dynamically in a data-driven way. Mapbox-gl is the primary platform supporting vector tiles, though there is some support in leafet (and others??). 

1. Upload to Mapbox
You can upload geojson (and shapefiles?) and mapbox will turn them into vector tiles which they also host. I don't use this pathway much though because there are limits on feature and file size and historically there was cost to serve tiles from mapbox (I think this changed in mapbox-gl v1).

2. Use tippecanoe
Tippecanoe is a tool to create vector tiles from geojson files. Input data _must_ be in EPSG:4326. Tippecanoe is powerful and dynamic and I use it for most tile export. 

3. Use gdal?
