
Spatial join
============

**Sources**

*Following materials are partly based on documentation of* `Geopandas <http://geopandas.org>`__.


`Spatial join <http://wiki.gis.com/wiki/index.php/Spatial_Join>`__ is
yet another classic GIS problem. Getting attributes from one layer and
transferring them into another layer based on their spatial relationship
is something you most likely need to do on a regular basis.

The previous materials focused on learning how to perform a `Point in Polygon query <Lesson3-point-in-polygon.html#how-to-check-if-point-is-inside-a-polygon>`__.
We could now apply those techniques and create our
own function to perform a spatial join between two layers based on their
spatial relationship. We could for example join the attributes of a
polygon layer into a point layer where each point would get the
attributes of a polygon that ``contains`` the point.

Luckily, `spatial join <http://geopandas.org/mergingdata.html#spatial-joins>`__
(``gpd.sjoin()`` -function) is already implemented in Geopandas, thus we
do not need to create it ourselves. There are three possible types of
join that can be applied in spatial join that are determined with ``op``
-parameter:

-  ``"intersects"``
-  ``"within"``
-  ``"contains"``

Sounds familiar? Yep, all of those spatial relationships were discussed
in the `previous materials <Lesson3-point-in-polygon.html>`__, thus you should know how they work.

Let's perform a spatial join between the address-point Shapefile that we
`created <Lesson3-table-join.html>`__ and then `reprojected <Lesson3-projections.html>`__
and a Polygon layer that is a
250m x 250m grid showing the amount of people living in Helsinki Region.

Download and clean the data
~~~~~~~~~~~~~~~~~~~~~~~~~~~

For this lesson we will be using publicly available population data from
Helsinki that can be downloaded from `Helsinki Region Infroshare
(HRI) <http://www.hri.fi/en/dataset/vaestotietoruudukko>`__ which is an
excellent source that provides all sorts of open data from Helsinki,
Finland.

From HRI **download a** `Population grid for year
2015 <https://www.hsy.fi/sites/AvoinData/AvoinData/SYT/Tietoyhteistyoyksikko/Shape%20(Esri)/V%C3%A4est%C3%B6tietoruudukko/Vaestotietoruudukko_2015.zip>`__
that is a dataset (.shp) produced by Helsinki Region Environmental
Services Authority (HSY) (see `this
page <https://www.hsy.fi/fi/asiantuntijalle/avoindata/Sivut/AvoinData.aspx?dataID=7>`__
to access data from different years).

-  Unzip the file in Terminal into a folder called Pop15 (using -d flag)

.. code:: bash

    $ cd
    $ unzip Vaestotietoruudukko_2015.zip -d Pop15
    $ ls Pop15
    Vaestotietoruudukko_2015.dbf  Vaestotietoruudukko_2015.shp
    Vaestotietoruudukko_2015.prj  Vaestotietoruudukko_2015.shx

You should now have a folder ``/home/geo/Pop15`` with files listed
above.

-  Let's read the data into memory and see what we have.

.. code:: python

    import geopandas as gpd
    
    # Filepath
    fp = r"/home/geo/Pop15/Vaestotietoruudukko_2015.shp"
    
    # Read the data
    pop = gpd.read_file(fp)
    
    # See the first rows
    print(pop.head())


.. parsed-literal::

       ASUKKAITA  ASVALJYYS  IKA0_9  IKA10_19  IKA20_29  IKA30_39  IKA40_49  \
    0          8       31.0      99        99        99        99        99   
    1          6       42.0      99        99        99        99        99   
    2          8       44.0      99        99        99        99        99   
    3          7       64.0      99        99        99        99        99   
    4         19       23.0      99        99        99        99        99   
    
       IKA50_59  IKA60_69  IKA70_79  IKA_YLI80  INDEX  \
    0        99        99        99         99    688   
    1        99        99        99         99    703   
    2        99        99        99         99    710   
    3        99        99        99         99    711   
    4        99        99        99         99    715   
    
                                                geometry  
    0  POLYGON ((25472499.99532626 6689749.005069185,...  
    1  POLYGON ((25472499.99532626 6685998.998064222,...  
    2  POLYGON ((25472499.99532626 6684249.004130407,...  
    3  POLYGON ((25472499.99532626 6683999.004997005,...  
    4  POLYGON ((25472499.99532626 6682998.998461431,...  
    

Okey so we have multiple columns in the dataset but the most important
one here is the column ``ASUKKAITA`` (*population in Finnish*) that
tells the amount of inhabitants living under that polygon.

-  Let's change the name of that columns into ``pop15`` so that it is
   more intuitive. Changing column names is easy in Pandas / Geopandas
   using a function called ``rename()`` where we pass a dictionary to a
   parameter ``columns={'oldname': 'newname'}``.

.. code:: python

    # Change the name of a column
    pop = pop.rename(columns={'ASUKKAITA': 'pop15'})
    
    # See the column names and confirm that we now have a column called 'pop15'
    print(pop.columns)


.. parsed-literal::

    Index(['pop15', 'ASVALJYYS', 'IKA0_9', 'IKA10_19', 'IKA20_29', 'IKA30_39',
           'IKA40_49', 'IKA50_59', 'IKA60_69', 'IKA70_79', 'IKA_YLI80', 'INDEX',
           'geometry'],
          dtype='object')
    

-  Let's also get rid of all unnecessary columns by selecting only
   columns that we need i.e. ``pop15`` and ``geometry``

.. code:: python

    # Columns that will be sected
    selected_cols = ['pop15', 'geometry']
    
    # Select those columns
    pop = pop[selected_cols]
    
    # Let's see the last 2 rows
    print(pop.tail(2))


.. parsed-literal::

          pop15                                           geometry
    5782      9  POLYGON ((25513499.99632164 6685498.999797418,...
    5783  30244  POLYGON ((25513999.999929 6659998.998172711, 2...
    

Now we have cleaned the data and have only those columns that we need
for our analysis.

Join the layers
~~~~~~~~~~~~~~~

Now we are ready to perform the spatial join between the two layers that
we have. The aim here is to get information about **how many people live
in a polygon that contains an individual address-point** . Thus, we want
to join attributes from the population layer we just modified into the
addresses point layer ``addresses_epsg3879.shp``.

-  Read the addresses layer into memory

.. code:: python

    # Addresses filpath
    addr_fp = r"/home/geo/addresses_epsg3879.shp"
    
    # Read data
    addresses = gpd.read_file(addr_fp)
    
    # Check the head of the file
    print(addresses.head(2))


.. parsed-literal::

                                     address  \
    0  Kampinkuja 1, 00100 Helsinki, Finland   
    1   Kaivokatu 8, 00101 Helsinki, Finland   
    
                                          geometry    id  
    0  POINT (25496123.30852197 6672833.941567578)  1001  
    1  POINT (25496774.28242895 6672999.698581985)  1002  
    

-  Let's make sure that the coordinate reference system of the layers
   are identical

.. code:: python

    # Check the crs of address points
    print(addresses.crs)
    
    # Check the crs of population layer
    print(pop.crs)
    
    # Do they match? - We can test that
    addresses.crs == pop.crs


.. parsed-literal::

    {'y_0': 0, 'no_defs': True, 'x_0': 25500000, 'k': 1, 'lat_0': 0, 'units': 'm', 'lon_0': 25, 'ellps': 'GRS80', 'proj': 'tmerc'}
    {'y_0': 0, 'no_defs': True, 'x_0': 25500000, 'k': 1, 'lat_0': 0, 'units': 'm', 'lon_0': 25, 'ellps': 'GRS80', 'proj': 'tmerc'}
    True



Indeed they are identical. Thus, we can be sure that when doing spatial
queries between layers the locations match and we get the right results
e.g. from the spatial join that we are conducting here.

-  Let's now join the attributes from ``pop`` GeoDataFrame into
   ``addresses`` GeoDataFrame by using ``gpd.sjoin()`` -function

.. code:: python

    # Make a spatial join
    join = gpd.sjoin(addresses, pop, how="inner", op="within")
    
    # Let's check the result
    print(join.head())


.. parsed-literal::

                                              address  \
    16  Malminkartanontie 17, 00410 Helsinki, Finland   
    19     Pitäjänmäentie 15, 00370 Helsinki, Finland   
    12       Trumstigen 8, 00420 Helsingfors, Finland   
    11           Kuparitie 8, 00440 Helsinki, Finland   
    15            Kylätie 23, 00320 Helsinki, Finland   
    
                                           geometry    id  index_right  pop15  
    16  POINT (25492349.68368251 6681772.551210108)  1017         2684     74  
    19  POINT (25492292.61413005 6679039.264838208)  1020         2691    241  
    12  POINT (25493207.62503373 6680836.727432437)  1013         2852    577  
    11  POINT (25493575.10327127 6679775.868274149)  1012         2949    562  
    15   POINT (25494077.1680778 6678341.639159317)  1016         3036    414  
    

Awesome! Now we have performed a successful spatial join where we got
two new columns into our ``join`` GeoDataFrame, i.e. ``index_right``
that tells the index of the matching polygon in the ``pop`` layer and
``pop15`` which is the population in the cell where the address-point is
located.

-  Let's save this layer into a new Shapefile

.. code:: python

    # Output path
    outfp = r"/home/geo/addresses_pop15_epsg3979.shp"
    
    # Save to disk
    join.to_file(outfp)

Do the results make sense? Let's evaluate this a bit by plotting the
points where color intensity indicates the population numbers.

-  Plot the points and use the ``pop15`` column to indicate the color.
   ``cmap`` -parameter tells to use a sequential colormap for the
   values.

.. code:: python

    import matplotlib.pyplot as plt
    
    joined.plot(c=joined['pop15'], markersize=6, cmap='Reds')
    plt.show()

.. image:: ../img/Lesson3-spatial-join_18_1.png

By knowing approximately how population is distributed in Helsinki, it
seems that the results do make sense as the points with highest
population are located in the south where the city center of Helsinki
is.
