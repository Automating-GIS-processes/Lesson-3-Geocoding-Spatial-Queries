# Exercise 3: Hints

More hints will be added here soon if needed. 


## Reading the csv-file with special characters (ä, ö)

if you get an error when reading in the shapefile with shopping center adresses ("UnicodeDecodeError: 'utf-8' codec can't decode byte.."), Try saving your text file again (File > Save As) in the computer instance and change the  character encoding to  'Current Locale (UTF-8)'

## Join attributes to the geocoded adresses

The merge-function will not work if the adresses in your textfile are not identical to those in Google Maps. If this is the case, you can use the pandas join: 

join = geo.join(data,lsuffix='_l', rsuffix='_r')

