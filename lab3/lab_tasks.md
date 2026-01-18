Exercises
1. Creating a Table & Measuring Distances
Use the spherical coordinate system WGS-84 (EPSG:4326).

Create a table cities with:

name (city name)
geom (GEOMETRY type, city center point)
Add at least two cities using coordinates from Wikipedia (via the “GeoHack” link). Remember:

Wikipedia lists coordinates as (latitude, longitude).

WKT format uses (longitude, latitude). Example for Kraków:

POINT(19.938333 50.061389)
Note: ST_GeomFromText() and ST_MakePoint() create geometries with SRID=0 by default. Use ST_SetSRID():

ST_SetSRID(ST_MakePoint(lon, lat), 4326)
or pass SRID directly to ST_GeomFromText().

Compute the distance between your two cities using ST_Distance():

SELECT ST_Distance(geom1, geom2);
Observe the units — they’re degrees, not meters.

To get meters:

Find a suitable projected coordinate system (search spatialreference.org).

Transform coordinates using ST_Transform():

ST_Transform(geom, <EPSG code>)
Then recalculate distance.

Also compute distance using ST_DistanceSphere() and ST_DistanceSpheroid(). Compare the results and record them in your report. Note that these

2. GEOMETRY vs. GEOGRAPHY
As noted here,

The GEOMETRY data type always assumes a flat plane — accurate only within a local coordinate system. In contrast, GEOGRAPHY works directly on the Earth’s ellipsoid (WGS-84), calculating distances and areas in meters.

Add a GEOGRAPHY column to your cities table.
Copy data from the GEOMETRY column, and cast to GEOGRAPHY.
Repeat Exercise 1 with the GEOGRAPHY data and compare results.
💡 Tip: GEOGRAPHY is slower but eliminates the need for projection — ideal for global data. Read more here.

3. Data Import
Download the dataset archive from this address.

The archive includes:

admin – administrative boundaries (multipolygon)
lamps – street lamps (points)
roads – roads (multilinestring)
You may place these tables in a separate schema, e.g. krakow. Remember to include it in your search_path:

SET search_path TO krakow, public;
Import data using the \i metacommand in psql:

\i admin.sql
or use psql -f … directly in the shell.

Check coordinate systems for all layers using ST_SRID():

SELECT ST_SRID(geom) FROM <table> LIMIT 1;
Record your findings in the report.

4. Area of Krakow
Compute Krakow’s area in square meters using both GEOMETRY and GEOGRAPHY.

Compare results with the official area listed on Wikipedia.

💡 Tip: Always use a projected coordinate system (like EPSG:2180) for accurate area measurements in meters.

5. Motorway Length
Find the total length of all motorways with road_type='motorway' that lie within Krakow’s administrative boundaries.

Hint: combine ST_Intersection() with ST_Length().

6. Street Lamps Near Czarnowiejska
Count lamps within 10 m of any road segment named Czarnowiejska.

Use ST_Buffer() to approximate road width and choose endcap options that minimize overlapping buffers at road intersections.

💡 Tip: A buffer width of 5–10 m usually works well. Use endcap='flat' to avoid rounded ends that double-count lamps.