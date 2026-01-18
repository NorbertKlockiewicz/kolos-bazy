# Advanced Database Systems - Lab 3 - Norbert Klockiewicz

## 1. Creating a Table & Measuring Distances

Create a table:
```
CREATE TABLE cities (
    name VARCHAR(100),
    geom GEOMETRY(POINT, 4326)
);
```

Insert Cities (New York and San Francisco):

```
INSERT INTO cities (name, geom) 
VALUES ('New York', ST_SetSRID(ST_MakePoint(-74.0, 40.716667), 4326));

INSERT INTO cities (name, geom) 
VALUES ('San Francisco', ST_SetSRID(ST_MakePoint(-122.419167, 37.779167), 4326));
```

Calculate Distance between cities:

```
nklockiewicz=> SELECT ST_Distance(
    (SELECT geom FROM cities WHERE name = 'New York'),
    (SELECT geom FROM cities WHERE name = 'San Francisco')
) AS distance_degrees;
 distance_degrees  
-------------------
 48.50819146519368
(1 row)
```

Calculating distance in meters:

```
SELECT ST_Distance(
    ST_Transform((SELECT geom FROM cities WHERE name = 'New York'), 2180),
    ST_Transform((SELECT geom FROM cities WHERE name = 'San Francisco'), 2180)
) AS distance_meters_projected;

 distance_meters_projected 
---------------------------
        5459695.940406718
```

Calculating distance with `ST_DistanceSphere`:

```
SELECT ST_DistanceSphere(
    (SELECT geom FROM cities WHERE name = 'New York'),
    (SELECT geom FROM cities WHERE name = 'San Francisco')
) AS distance_meters_sphere;

 distance_meters_sphere 
------------------------
       4129317.86341195

```

Calculating distance with `ST_DistanceSpheroid`:

```
SELECT ST_DistanceSpheroid(
    (SELECT geom FROM cities WHERE name = 'New York'),
    (SELECT geom FROM cities WHERE name = 'San Francisco'),
    'SPHEROID["WGS 84",6378137,298.257223563]'
) AS distance_meters_spheroid;

 distance_meters_spheroid 
--------------------------
       4139373.0499254894
```

### Analysis

What's interesting about different types of queries which are measuring distance is that for each we got different results, here's why:
- projection to meters - it transforms coordinates to a flat, 2D projected coordinate system and uses cartesian distance. It's great where we are measuring distances between close points/
- `ST_DistanceSphere` - it assumes that Earth is a perfect sphere which gives as more accurate results
- `ST_DistanceSpheroid` - the most accurate, it assumes that Earth is flattened at poles

To sum up results shown that the projection to meters is way off and the results for `ST_DistanceSphere` and `ST_DistanceSpheroid` are close to each other

## 2. GEOMETRY vs. GEOGRAPHY

Add a GEOGRAPHY column to cities table:

```
ALTER TABLE cities ADD COLUMN geog GEOGRAPHY(POINT, 4326);
```

Copy data from GEOMETRY to GEOGRAPHY:

```
UPDATE cities SET geog = geom::GEOGRAPHY;
```

Calculate distance:

```
SELECT ST_Distance(
    (SELECT geog FROM cities WHERE name = 'New York'),
    (SELECT geog FROM cities WHERE name = 'San Francisco')
) AS distance_meters_geography;
 distance_meters_geography 
---------------------------
          4139373.04992549
```

### Analysis

The results is identical to one got by using `ST_DistanceSpheroid` however the fact that GEOGRAPHY by default that Earth is an Spheroid allows us to omit the need of transformation.

## 3. Data Import

Create Schema and import data:

```
CREATE SCHEMA IF NOT EXISTS krakow;
SET search_path TO krakow, public;

\i lab3/admin.sql

\i lab3/lamps.sql

\i lab3/roads.sql
```

Check the SRID:

| Table  | SRID | Coordinate System |
|--------|------|-------------------|
| admin  | 2180 | ETRS89 / Poland CS92 |
| lamps  | 2178 | ETRS89 / Poland CS 2000 zone 7 |
| roads  | 4326 | WGS 84 (Geographic) |

Each table has different SRID so that means that we will need to use transformation while working with this data.

## 4. Area of Krakow

Calculating using GEOMETRY:

```
SELECT ST_Area(geom) AS area_m2_geometry
FROM admin
WHERE name = 'Kraków';

 area_m2_geometry  
-------------------
 326399267.4317564
(1 row)
```

Calculating using GEOGRAPHY:

```
SELECT ST_Area(ST_Transform(geom, 4326)::GEOGRAPHY) AS area_m2_geography
FROM admin
WHERE name = 'Kraków';
 area_m2_geography  
--------------------
 326816324.74502707
```

Tip: We needed to transform to different coordinate system

Comparision:

Wikipedia: 326,85 km²
Our queries: 326,8 and 326,4

## 5. Motorway Length

Get the motorways length:

```
SELECT SUM(ST_Length(ST_Intersection(ST_Transform(r.geom, 2180), a.geom))) AS total_motorway_length_m
FROM roads r
JOIN admin a ON ST_Intersects(ST_Transform(r.geom, 2180), a.geom)
WHERE a.name = 'Kraków' AND r.road_type = 'motorway';

 total_motorway_length_m 
-------------------------
       32699.44034178033
```

## 6. Street Lamps Near Czarnowiejska

Get the lamps count:

```
SELECT COUNT(*) AS lamp_count
FROM lamps l
JOIN roads r ON r.road_name = 'Czarnowiejska'
WHERE ST_Contains(
    ST_Buffer(ST_Transform(r.geom, 2180), 10, 'endcap=flat'), 
    ST_Transform(l.geom, 2180)
);

 lamp_count 
------------
         92
(1 row)
```