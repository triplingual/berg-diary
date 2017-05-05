# Locating Mary Berg's Diary
Final project for HIS 575 at Southern Connecticut State University, Spring 2016, Professor Troy Paddock

## "Locating Mary Berg's Diary" Workflows

### GeoJSON Processing

Get GeoJSON data or get data into GeoJSON

I made the choice to use modern boundaries and used [a Polish government site](http://www.codgik.gov.pl/index.php/darmowe-dane/prg.html) for Ciechocinek, Łódź, Lowicz, and Sochaczew, then found [Mapzen's metro extracts service](https://mapzen.com/data/metro-extracts/) and used it for Lisboa/Lisbon, Metz, Biarritz, Nancy, Zbaszn (changed to Zbąszynek after the war), and Saarbrücken.

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

Use it to get the centroid of a city's polygon with the recipe at http://pcjericks.github.io/py-gdalogr-cookbook/geometry.html?highlight=centroid#quarter-polygon-and-create-centroids

tl;dr: 
```
import ogr
# Given a test polygon
poly_wkt= "POLYGON((LON LAT,LON LAT,etc.))"
geom_poly = ogr.CreateGeometryFromWkt(poly_wkt)
centroid = geom_poly.Centroid()
print centroid    # get the value pair from this output
```