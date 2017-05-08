# Locating Mary Berg's Diary
Final project for HIS 575 at Southern Connecticut State University, Spring 2016, Professor Troy Paddock

## "Locating Mary Berg's Diary" Workflows

### Setting up GDAL on Mac OS X

Use the KyngChaos binary to install GDAL

Some people use Homebrew. Out of inertia, concern for running non-Homebrew stuff in parallel with Homebrew, and not knowing the consequences of installing Homebrew on a machine that's had a lot installed without it, I don't. But people have documented [installing GDAL with Homebrew](https://mountainsol.wordpress.com/2014/08/13/install-gdal-on-mac-os-x-mavericks/)

[Add GDAL to your PATH](https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path)

[clhenrick on GitHub](https://github.com/clhenrick/)  has some [nifty shell scripts](https://github.com/clhenrick/shell_scripts) for doing some things with GDAL quickly.

### GeoJSON Processing

Get GeoJSON data or get data into GeoJSON

I made the choice to use modern boundaries and used [a Polish government site](http://www.codgik.gov.pl/index.php/darmowe-dane/prg.html) for Ciechocinek, Łódź, Lowicz, and Sochaczew, then found [Mapzen's metro extracts service](https://mapzen.com/data/metro-extracts/) and used it for Lisboa/Lisbon, Metz, Biarritz, Nancy, Zbaszn (changed to Zbąszynek after the war), and Saarbrücken.

The Polish government files were ESRI shapefiles, so I needed to convert them with GDAL, making sure to set the output to UTF-8 as they were previously in Windows-1250 encoding (understandable given the source, but really, folks, do everything in UTF-8 if you have any control at all over that):

```
export SHAPE_ENCODING="Windows-1250"
ogr2ogr -f GeoJSON -t_srs crs:84 gminy-municipalities.geojson gminy.shp -lco ENCODING=UTF-8
```

I found a complete set of French commune boundary files, but it was in MapInfo format, so I used ogr2ogr to transform them. Simple format change was not terribly complicated:

`ogr2ogr -f GeoJSON [target datafile].tab [source datafile]`

But then it turned out that they used RGF93 /Lambert-93 projection, which was initially beyond my ken. To the Internet! And I found [a helpful blog post from Frank Donnelly of Baruch College](http://gothos.info/2009/04/transform-projections-with-gdal-ogr/) describing what to do.

`ogr2ogr -f GeoJSON -t_srs EPSG:4326 communes-00.geojson COMMUNE.tab`

#### Validation

It's a good idea to validate your GeoJSON syntax. In this project, I tended to use [JSONLint](https://jsonlint.com/). There's probably a shell utility that will do that, if you're strictly a shell person.

Though it's not validation, strictly speaking, I also used [JSON Pretty Print](http://jsonprettyprint.com/json-pretty-printer.php) to neaten things up. Helps make it more readable.

#### Cities

Use this regex to cut the number of points in a complex polygon in half. I don't need the city boundaries to be more accurate than that. For Łódź I cut it down into 1/4 or maybe even 1/8 of the original number of points.

Find:

	(\[ -?\d{1,2}\.\d{9,15}, \d{1,2}\.\d{9,15} \],) \[ -?\d{1,2}\.\d{9,15}, \d{1,2}\.\d{9,15} \],

Replace:

	\1
	
Then strip the prefatory JSON until all you have are the coordinates for the polygon and the brackets and use this regex to format the pairs:

Find:

	\[ (-?\d{1,2}\.\d{9,15}), (\d{1,2}\.\d{9,15}) \],

Replace:

	\1 \2,

Do what you need to do to end up with a string that looks like

	"POLYGON((LON LAT,LON LAT,etc.))"

Use it with Python to get the centroid of a city's polygon with a variation of [Jared Erickson's create centroids recipe](http://pcjericks.github.io/py-gdalogr-cookbook/geometry.html?highlight=centroid#quarter-polygon-and-create-centroids) in his [Python+GDAL/OGR cookbook](http://pcjericks.github.io/py-gdalogr-cookbook/).

tl;dr: 
```
import ogr
poly_wkt= "[string from above text manipulations]"
geom_poly = ogr.CreateGeometryFromWkt(poly_wkt)
centroid = geom_poly.Centroid()
print centroid    # get the value pair from this output
```