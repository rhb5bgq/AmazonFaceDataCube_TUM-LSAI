README File For Amazon FACE Data Cube 

Version: Landsat 8 and Sentinel 2
16.07.2024
Ross Brown 
Other questions that aren't answered here can be directed to my email ross.brown.virginia@gmail.com

main package : GDAL Cubes - A way of making large data cubes in R that doesn't (always) crash your computer :D

------------------------ Supported Data ------------------------
collection_formats()
This command and function lists supported data products that work out of the box with GDAL cubes 
   - Landsat 8 is not on this list of supported files 
   - Sentinel 2 is a supported collection format 

When you have a data product that you would like to add to the cube but isn't a supported collection format, you have to create your own. A collection format is a java script notation script that you can create in notepad or in R. It tells GDAL cubes how to extract key information like temporal, spectral, and spatial information to organize into an image collection. 

Here is the script for Landsat 8: 
{
  "description" : "Collection format for Landsat 8 surface reflectance product (Edited by Ross Brown)",
  "references" : "https://landsat.usgs.gov/sites/default/files/documents/lasrc_product_guide.pdf",
  "tags" : ["Landsat", "USGS", "Level 2", "NASA", "surface reflectance"],
  "pattern" : ".+\\.tif",
  "images" : {
    "pattern" : ".*(L[OTC]08_.{4}_.{6}_.{8}_.{8}_.{2}_.{2})[A-Za-z0-9_]+\\.tif"
  },
  "datetime" : {
    "pattern" : ".*L[OTC]08_.{4}_.{6}_(.{8})_.*\\.tif",
    "format" : "%Y%m%d"
  },
  "bands": {
    "B01" : {
      "pattern" : ".+SR_B1\\.tif",
      "nodata" : -9999
    },
    "B02" : {
      "pattern" : ".+SR_B2\\.tif",
      "nodata" : -9999
    },
    "B03" : {
      "pattern" : ".+SR_B3\\.tif",
      "nodata" : -9999
    },
    "B04" : {
      "pattern" : ".+SR_B4\\.tif",
      "nodata" : -9999
    },
    "B05" : {
      "pattern" : ".+SR_B5\\.tif",
      "nodata" : -9999
    },
    "B06" : {
      "pattern" : ".+SR_B6\\.tif",
      "nodata" : -9999
    },
    "B07" : {
      "pattern" : ".+sr_SR_B7\\.tif",
      "nodata" : -9999
    },
    "PIXEL_QA" : {
      "pattern" : ".+_QA_PIXEL\\.tif"
    },
    "RADSAT_QA" : {
      "pattern" : ".+_QA_RADSAT\\.tif"
    },
    "AEROSOL" : {
      "pattern" : ".+QA_AEROSOL\\.tif"
    },
    "NDVI" : {
      "pattern" : ".+SR_NDVI\\.tif",
      "nodata" :-9999
    },
    "EVI" : {
      "pattern" : ".+SR_EVI\\.tif",
      "nodata" :-9999
    },
    "ST_B10" : {
      "pattern" : ".+ST_B10\\.tif",
      "nodata" :-9999
    },
    "EMIS" : {
      "pattern" : ".+ST_EMIS\\.tif",
      "nodata" :-9999
    },
    "EMSD" : {
      "pattern" : ".+SR_EMSD\\.tif",
      "nodata" :-9999
    },
    "ST_QA" : {
      "pattern" : ".+ST_QA\\.tif",
      "nodata" :-9999
    },
    "ST_TRAD" : {
      "pattern" : ".+ST_TRAD\\.tif",
      "nodata" :-9999
    }
  }
}


You can follow this pattern to build your own collection formats for use on other data products. I tried this one for the Modis GPP product, but something is not quite right in the collection format. Anja might want this added eventually. 

{
  "description" : "Collection format for selected bands from the MODIS 17A2H v061 product",
  "tags" : ["MODIS", "GPP", "NPP"],
  "pattern" : ".*\\.hdf.*",
  "subdatasets" : true,
  "images" : {
    "pattern" : "HDF4_EOS:EOS_GRID:\"(.+)\\.hdf.*"
  },
  "datetime" : {
    "pattern" : ".*M[OY]D17A2H\\.A(.{7}).*",
    "format" : "%Y%j"
  },
  "bands" : {
    "GPP" : {
      "pattern" : ".+Gpp_500m.*",
      "nodata" : -3000
    },
    "NetPhotosn" : {
      "pattern" : ".+PsnNet_500m.*",
      "nodata" : -3000
    },
    "NetPhotosnQA" : {
      "pattern" : ".+Psn_QC_500m.*",
      "nodata" : -3000
    }
  }
}


------------------------ Necessary Steps in Creating A Data Cube Steps ------------------------

1. Creating an image collection: 

Once you have found your target data and defined your filepaths, either to data in the cloud or physically downloaded, you can create an image collection this is done with 

create_image_collection() - this function will create an image collection from a physically downloaded image data set 

stac_image_collection() - this function can query STAC (libary(rstac)) for an image collection for a specified area, time, and cloud cover scene %. I used this for sentinel 2, but I found that it can be used for Landsat 8 as well. This should probably be integrated to create easier data fusion in the future. 

image_collection() - this function can create an image collection out of an existing rastercube or exported netcdf. More on raster cubes and exported netcdf's later. 

2. Creating a cube view: 

cube_view() - this function allows you to define projections, image collections, temporal extent, spatial extent, spatial resolution, and the aggregation and resampling functions needed to reduce the datacube i.e. the geometry of a cube without connecting it to any data. 

The dt argument is how often you want your cube reduced i.e. dt = "P1M" would reduce your cube into monthly averages, medians, or whatever other operator you would like to apply.

dt = "P1D" would reduce it into individual days 

3. Creating a mask: 

image_mask() is a functions where you define the quality control band and assign values to filter out. This is essential for filtering out any leftover clouds, atmospheric haze, and cloud shadows 

4. Creating a raster cube: 

raster_cube() - Having defined an image collection, and a data cube view, a data cube is simply the combination of the two. You can create a data cube with the raster_cube() function 

you can either define a raster cube and refrence it in plots later or pipe a raster cube with functions so that you get an end result with one collective code block. Here are some common functions that I used when piping raster cubes: 

- apply_pixel():
Usage: Applies a pixel-wise function or expression to a data cube.
Situation: Use this function to compute derived data (e.g., vegetation indices like NDVI or EVI) from the bands in your data cube.

- reduce_time():
Usage: Applies a reduction operation (e.g., mean, sum, max) over the time dimension of a data cube.
Situation: Use this function to summarize data over time, such as calculating monthly or yearly averages.

- aggregate_time():
Usage: Aggregates data over the time dimension using a specified time period (e.g., daily, monthly).
Situation: Use this function to resample your data cube to a different temporal resolution, like converting daily data to monthly summaries.

- reduce_space():
Usage: Applies a reduction operation (e.g., mean, sum, max) over the spatial dimensions of a data cube.
Situation: Use this function to summarize data across the spatial dimensions, such as computing average values across a region.

- plot():
Usage: Plots a data cube or its slices.
Situation: Use this function to visualize the contents of your data cube, helping to understand spatial and temporal patterns. I used this for time series and image plots, but I recommend exporting to a netcdf, creating a tibble or data frame and using other packages like tmap or ggplot to create figures since the plot function with GDAL cubes is very stylistically limiting 

- write_ncdf():
Usage: Writes a data cube to a NetCDF file.
Situation: Use this function to export your data cube to a widely-used file format for sharing, archiving, or further analysis in other software or packages.

- animate():
Usage: Creates an animation of a data cube over time.
Situation: Use this function to create temporal animations of your data, useful for presentations or dynamic visualizations.

- write_tif():
Usage: Writes a data cube to a GeoTIFF file.
Situation: Use this function to export specific slices or summaries of your data cube to a GeoTIFF format for use in GIS applications.

- gdalcubes_options():
Usage: Configures global options for the gdalcubes package.
Situation: Use this function to set options like the number of threads for parallel processing, affecting the performance of data cube operations.

- select_bands(): 
Usage: Selects specific bands from a data cube.
Situation: Use this function to create a new data cube containing only a subset of bands from the original data cube, which is useful for focusing on specific spectral information.

- filter_geom():
Usage: Filters a data cube by a spatial geometry (e.g., a polygon).
Situation: Use this function to subset a data cube to only include data within a specified spatial geometry, such as a region of interest.


