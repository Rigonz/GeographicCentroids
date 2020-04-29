# GeographicCentroids
Geographic centroids with population weighing

## Presentation
As far as one can judge, there are some common errors in the **calculation of the centroids of geographical areas**. They are not huge indeed, but they are easily preventable: this project presents the issues and one of the existing solutions.

Also, I am not aware of a **repository of geographical centroids**, even less if they are **weighed by population**. This project provides one such repository.

Finally, it includes a **Jupyter Notebook** that can be easily adapted. It uses a very large public dataset of populated places to calculate different centroids with two pproaches:
1. On one side the purely geographic centroids but also weighed by population: the dataset includes detailed population estimates for the years 2000, 2005, 2010, 2015 and 2020, so it is possible to compute the corresponding centroids (and compare their differences). 
2. On the other side, this dataset allows different degrees of administrative aggregation: nation, estate, province, etc. Two sets of results are provided: at the administrativel level 1 (248 nations) and at level 2 for certain large countries (CHN, RUS, USA, AUS, CAN and BRA).

## Basics: Physics and  Cartography
The centroid of a set of points (or a flat surface or a volume) is defined in physics textbooks as the point which minimizes the total distance from the set of points to the centroid. In a cartesian coordinates system and for the usual euclidean distance the centroid is found by averaging the (x, y, z) coordinates of the points. If a weighing has to be applied (for instance the population asigned to the points) the formulas are factored by the corresponding weigh. (See f.eg.[Centroid in Wikipedia](https://en.wikipedia.org/wiki/Centroid)).

These averaging and factoring expressions are quite convenient for calculation. However, they are valid for a cartesian system of coordinates, and they are **not** for immediate application with geographical reference systems (longitude-latitude, spherical coordinates). 

A simple example can show this for a pair of points whose geographical coordinates are:
* P1 = (0,  0)
* P2 = ( 20, 20)

The centroid calculated by the average formula is located at (10, 10). However, the geodetic distances from P1 and P2 to this "centroid" are  (assuming the coordinate pairs are in lon-lat, not lat-lon):
* P0-P1 = 1565.1  km
* P0-P2 = 1541.9 km

As calculated with the following script:

    from geopy.distance import geodesic
    P1  = [0,   0]
    P2  = [20, 20]
    P0  = [10, 10]
    D01 = geodesic(P0, P1).km
    D02 = geodesic(P0, P2).km
    print (D01, D02)

A great-circle calculator would provide a differente value. For example, [NOAA](https://www.nhc.noaa.gov/gccalc.shtml) yields 1567 km and 1544 km. Other pages give different results: 1569 and 1545 km from [LATLONG.](https://www.movable-type.co.uk/scripts/latlong.html)

The mistake of averaging spherical coordinates to obtain the centroid seems not to be uncommon, as presented in the following two examples:
1. [Baylor](https://cs.baylor.edu/~hamerly/software/europe_population_weighted_centers.html)

        select countrynm,
            (sum(lat_cen * p00a) / sum(p00a)) as latitude,
            (sum(long_cen * p00a) / sum(p00a)) as longitude, 
            sum(p00a) as population
        from centroids 
        group by countrynm;
  
2. [Sumit](https://medium.com/@sumit.arora/plotting-weighted-mean-population-centroids-on-a-country-map-22da408c1397)

        points = MultiPoint(geoList)
        result_df = result_df.append({'State':df['State'].iloc[index-1],'District':df['District'].iloc[index-1],'Latitude':points.centroid.x,'Longitude':points.centroid.y}, ignore_index=True)
The library used in this second case, shapely, calculates the centroid as the mean of the given coordinates, without consideration to the coordinate units.

## Solutions
As far as I can see there are two alternatives for the correct calculation of these centroids: 
1. To continue in a cartesian, rectangular system of coordinates. 
2. Adjust the units of the longitude.

Alternative 2 is used by the US Census Bureau as described here: [Census.](https://www2.census.gov/geo/pdfs/reference/cenpop2010/COP2010_documentation.pdf)

If we want to avoid mixing angle and linear units the conversion is:

    x = a*cos(LAT)*cos(LON)
    y = a*cos(LAT)*sin(LON)
    z = b*sin(LAT)
Where a and b are the semiaxis of the selected geoid.
The centroid will not lay on the surface but within the volume of the geoid. Its reprojection is straightforward from the previous equations:

    LAT = arcsin(Z/b)
    LON = arcsin (Y/a/cos(LAT))
Where Y and Z are the averages (weighed if necessary) of the y's and z's from the dataset.

## Dataset
The dataset used in this project is the "Administrative Unit Center Points with Population Estimates" v4.11 from GPW (Gridded Population of the World) at [SEDAC_GPW](https://sedac.ciesin.columbia.edu/data/set/gpw-v4-admin-unit-center-points-population-estimates-rev11).

The database set (which is very large, close to 2 GB) has been prepared with the following tasks:
* clearing parsing errors (I cannot say if they are on the original dataset or appeared during the file download, but I had to clear them),
* removing unused fields, as indicated in the Notebook, 
* merging the four datasets from USA.

## Jupyter Notebook
See the two files included in this repository.

## Results
The results are included in the files:
* "country centroids R0.csv"
* "region centroids R0.csv"

An example of the differences between geographic and demographic centroids, and the displacement of the demographic centroid in the last 20 years can be seen in the repository: [pics](https://github.com/Rigonz/GeographicCentroids/blob/master/pics/).

## Warning
The geographic centroid of Russia seems to be wrong, all other countries seem good. Will have to look.

![countries](/pics/GeoCentroids.png)
![regions](/pics/RegionCentroids.png)
