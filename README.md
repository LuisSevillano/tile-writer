Slippy map tile generator for QGIS   
Copyright 2015 **Alexander Hajnal**

http://alephnull.net/software/gis/tile_writer.shtml

_This is an updated versión of the original script to work with Python 3 and QGIS 3 (Oct 2018)_

# About

This program is a slippy map tile generator for use with QGIS. All rendering is
done by QGIS so what you see on your screen is what you'll get in your output
tiles. Functionally it is quite similar to the [QTiles plugin](https://github.com/nextgis/QTiles). Here's a
comparison of the two:

Advantages of this program:
 - Renders the map a section at time.
   This allows even very large datasets to be rendered.
 - Renders each zoom level separately. This means that scale-dependant feature visibility is honored.
 - Does not exhibit the rendering issues that QTiles has at tile edges:
   - Does not truncate point icons.
   - Labels don't shift.
   - Patterned lines display correctly.
   - Does not in QGIS 3.
 - Fast: 20 minutes to render a complex 60km x 70km topographic map with 
   roads, water, POIs, etc. from levels 5 through 15 on a 3.5GHz i7.
 - Resistant to the QGIS bug that causes raster layers to sometimes not be 
   displayed.
 - Copious progress status display
 - Simple Python script that can be easily modified to suit your needs.

Disadvantages of this program:
 - No graphical interface.
 - Cannot be made to stop early without killing QGIS.
 - Rendering complex maps causes the QGIS GUI to temporarily freeze during 
   some parts of the rendering process.
 - Lacks some of the additional options that QTiles offers.
 - Doesn't always clip the rendering to the specified area of interest.
   At lower zoom levels tiles will be rendered that are not within the area 
   of interest.

Comments are welcome, you can email SLIPPYsoftware@alephnull.net
(remove the type of map to get the real address).


## Version history
| Version | Comment     |
| :------------- | :------------- |
| **v0.3**       | Update to Work with Python 3 in QGIS 3.       |
| **v0.2.1**       | Documentation updates, added scale presets file (no code changes).      |
| **v0.2**       | Now supports both TMS and Google naming conventions.       |
| **v0.1**       | Initial release.       |

## Files

| File | Description |
| :------------- | :------------- |
| `readme.md`         | This file |
| `tile_writer.py`     | The main script |
| `globalmercator.py`  | For latitude/longitude to tile coordinate conversions Taken from GDAL2Tiles |
| `scales.xml `        | Scale presets for QGIS matching the tile zoom levels |


## Usage

Prelimary steps:
 - Create your map in QGIS
   - If you are using scale-dependant feature visibility see the section [**Zoom levels**](#zoom-levels).

Running the script:
 - Place `tile_writer.py` and `globalmercator.py` into the same directory
   (As packaged, this has already been done).
 - In QGIS, open the Python console (Plugins => Python Console).
 - Click on the "Show editor" icon in the Python Console (middle icon).
 - Load the `tile_writer.py` script using the "Open file" icon (top icon on the right).
 - Define the map's area of interest:
   - Create a new polygon vector layer.
   - Make it editable.
   - Add a polygon covering the area you want rendered.
   - Save it to disk; the filename you chose should be put into the .
     area_of_interest variable in the script.
   - Hide the layer.
 - Adjust the settings to suit your needs (see below).
 - Note that all paths and filenames are relative to QGIS's current working 
   directory.
   - If you run QGIS from the commandline then the directory you started it 
     from is the working directory.
   - If you run QGIS from a graphical interface under Unix/Linux then the 
     working directory is probably your home directory.
   - If you run QGIS under Windows drop me an email and let me know what 
     QGIS's working directory is :^).
 - Run the script using the "Run script" icon (the bottom icon).
 - The script's progress will appear in the console.
 - An "All done" message will be printed when the script has finished.

If you have difficulty getting the globalmercator module to load or any error try insert into the QGIS Python console:
```python
# https://gis.stackexchange.com/a/283468/90253
import sys
sys.path.append("YOUR-PATH-HERE/tile_writer")
```

Note that QGIS may appear to have locked up while generating the regional tiles.
If this happens simply be patient; complex maps may take several minutes to 
draw.

The script will avoid regenerating images whenever possible.  This means that 
if you want to regenerate a map you should either specify a different 
`output_path` or first delete all of the tile directories (numbered) as well as the region tiles (e.g. `10_300_651_s16_b2.png`).


## Configuration variables

These are found near the top of `tile_writer.py` and should be changed as appropriate before running the script.

<a href="#start_z" name="start_z">#</a><b> start_z</b>: Minimum zoom level. Default value: `10`.

<a href="#end_z" name="end_z">#</a><b> end_z</b>: Maximum zoom level. Default value: `15`.

<a href="#step" name="step">#</a><b> step</b>: How many map tiles to place in each regional tile
  (the actual number of map tiles per regional tile is step * step) 
  Higher number speed up rendering (assuming enough RAM is available).
  Lower numbers decrease memory usage but increase rendering time.
  Default value: `16`.

<a href="#border" name="border">#</a><b> border</b>: Number of extra map tiles to render along each edge of a regional tile. By rendering extra, unused border tiles we can avoid shifting labels, truncated images, and line-drawing inconsistancies at tile boundaries. If this value is set to 0 you will encounter rendering issues at tile boundaries. Default value: `2`.

<a href="#output_path" name="output_path">#</a><b> output_path</b>: Directory to write the regional images and level subdirectories to. Default value: '.'.

<a href="#area_of_interest" name="area_of_interest">#</a><b> area_of_interest</b>: Path of shapefile defining the area of interest. The extent of the shapefile is used to limit rendering to a particular area. Note that no clipping is done so at lower zoom levels tiles from outside the area of interest will be generated. Default value: `border.shp`. You can extract the bounding box of a polygon in <kbd>Menu</kbd> -> <kbd>Vector</kbd> -> <kbd>Research Tools</kbd> -> <kbd>Polygon from layer extent</kbd>.

## Filename convention to follow
- Can be set to either `tms` or `google`.
- Default value: `tms`.


## Zoom levels

If you are using scale-dependant feature visibility you should use the scales 
listed below when deciding visibility.
These scales can be found in the scales.xml file and installed in QGIS using:
<kbd>Settings</kbd> -> <kbd>Options</kbd> -> <kbd>Map Tools</kbd> -> <kbd>Predefined Scales</kbd> -> <kbd>Import from file</kbd> (folder icon).

    Level Scale
    ----- ---------------------
    1     1 : 295,829,355.45
    2     1 : 147,914,677.73
    3     1 : 73,957,338.86
    4     1 : 36,978,669.43
    5     1 : 18,489,334.72
    6     1 : 9,244,667.36
    7     1 : 4,622,333.68
    8     1 : 2,311,166.84
    9     1 : 1,155,583.42
    10    1 : 577,791.71
    11    1 : 288,895.85
    12    1 : 144,447.93
    13    1 : 72,223.96
    14    1 : 36,111.98
    15    1 : 18,055.99
    16    1 : 9,028.00
    17    1 : 4,514.00
    18    1 : 2,257.00
    19    1 : 1,128.50
    20    1 : 564.25
    21    1 : 282.12
    22    1 : 141.06
    23    1 : 70.53

# Rendering time
After several tests:
- Render a square of 9km2, with levels of zoom between 12 and 17 takes aprox. 5 minutes.
- Render a square of 1.095km2 (Extent of Madrid city boundary) with levels of zoom between 12 and 17 takes aprox. 27 minutes.


# Warranty

I make no warranty or representation, either express or implied, with respect
the behavior of this script, its quality, performance, accuracy,
merchantability, or fitness for a particular purpose. This script is provided
'as is', and you, by making use thereof, are assuming the entire risk. That
said, I hope you this script useful. Have fun!
