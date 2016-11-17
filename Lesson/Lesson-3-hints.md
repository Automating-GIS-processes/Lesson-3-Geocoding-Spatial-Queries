# Exercise 3: Hints

More hints will be added here soon if needed. 


## Problem 1: 
### Reading the csv-file with special characters (ä, ö)

If you get an error when reading in the shapefile with shopping center adresses `("UnicodeDecodeError: 'utf-8' codec can't decode byte..")`, Try saving your text file again (File > Save As) in the computer instance and change the character encoding to  `Current Locale (UTF-8)`

### Join attributes to the geocoded adresses

The merge-function will not work properly if the adresses in your textfile are not identical to those returned by the geocoder. If this is the case, you can use the pandas join-function that joins the tables based on index values in both of the layers: 

```python
 join = geo.join(data, lsuffix='_l', rsuffix='_r')
```

If doing so, you might also want to drop extra columns and rename them such as following:

```
# Drop an extra column called 'id_r'
join = join.drop('id_r', axis=1)

# Rename a column 'id_l' as 'id'
join = join.rename(columns={'id_l': 'id'}
```

## Problem 2:

### Calculating the buffer

Instead of iterating over the rows in your GeoDataFrame there is also another way ('a shortcut') of calculating e.g. the area of Polygons, take following example:

```python

# Calculate the areas of polygons into a column called 'poly_area'
data['poly_area'] = data.area
```

What Geopandas does in the background is basically the same thing, i.e. it iterates over the rows and calculates the area for each row (in this case). You can use similar approach 
for doing the same thing with `buffer` function. 

### Specifying the column that is used as a source for geometries in GeoDataFrame

When having multiple geometries in a same GeoDataFrame (such as Points and Polygons), it is **necessary to drop either one of them when saving the GeoDataFrame into a Shapefile**. 
In other words, it means that you can only have a single column having Shapely geometries when saving the GeoDataFrame into a Shapefile. 

You can specify which column should be used as a source for geometries by using a function `.set_geometry()`. Let's continue the previous example and set the `poly_area` column as our geometry source-column:

 ```python
 # Define the source for geometries in a GeoDataFrame
 data = data.set_geometry('poly_area')
 
 # Let's see what is the current active geometry column
 print(data.geometry.name)
 ```
 
If we would like to save our GeoDataFrame, we still need to drop the original `geometry` column as a Shapefile cannot contain two geometry columns with Shapely objects:
  
 ```python
 # Drop a column ´geometry
 data = data.drop('geometry', axis=1)
 ```
 
## Problem 3
 
### Grouping data
Use the .groupby() function for grouping data. More information can be found under Lesson 2 Pro tips:
[Grouping data] (https://automating-gis-processes.github.io/2016/Lesson2-geopandas-basics.html#grouping-data)
