# Lab 3: PostGIS/Spatial Databases - Practice Questions

## Question 1: Distance Calculations (Similar to Test Question 1, part 1)

### Given the following table:

```sql
CREATE TABLE Cities (
    CityID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Location GEOMETRY(POINT, 4326) -- WGS-84
);

INSERT INTO Cities (Name, Location)
VALUES
('Warsaw', ST_SetSRID(ST_MakePoint(21.0122, 52.2297), 4326)),
('Krakow', ST_SetSRID(ST_MakePoint(19.9445, 50.0647), 4326)),
('Berlin', ST_SetSRID(ST_MakePoint(13.4050, 52.5200), 4326)),
('Prague', ST_SetSRID(ST_MakePoint(14.4378, 50.0755), 4326));
```

### Tasks:

1. **Write a query** to calculate the distance between Warsaw and Krakow in meters using:
   - ST_Transform() with EPSG:2180 (Poland coordinate system)

2. **Write a query** to calculate the same distance using ST_DistanceSpheroid()

3. **Explain** the difference between using GEOMETRY with ST_Transform() vs using ST_DistanceSpheroid()

---

## Question 2: Area Calculations (Similar to Test Question 1)

### Given:

```sql
CREATE TABLE Regions (
    RegionID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Boundary GEOMETRY(POLYGON, 4326)
);

INSERT INTO Regions (Name, Boundary)
VALUES
('Region1', ST_SetSRID(ST_GeomFromText('POLYGON((19 50, 22 50, 22 52, 19 52, 19 50))'), 4326)),
('Region2', ST_SetSRID(ST_GeomFromText('POLYGON((18 52, 21 52, 21 55, 18 55, 18 52))'), 4326));
```

### Tasks:

1. **Write a query** to compute the area of each region in square kilometers

2. **Explain** why you need to use ST_Transform() and which EPSG code you would use

3. **Write the complete query** with proper transformation to EPSG:2180

---

## Question 3: Spatial Relationships (Similar to Test Question 1, part 2)

### Given the tables from Question 2 (Regions) and:

```sql
CREATE TABLE PointsOfInterest (
    POI_ID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Type VARCHAR(50),
    Location GEOMETRY(POINT, 4326)
);

INSERT INTO PointsOfInterest (Name, Type, Location)
VALUES
('Museum A', 'Museum', ST_SetSRID(ST_MakePoint(20.5, 51.0), 4326)),
('Park B', 'Park', ST_SetSRID(ST_MakePoint(19.5, 53.0), 4326)),
('Library C', 'Library', ST_SetSRID(ST_MakePoint(21.5, 51.5), 4326));
```

### Tasks:

1. **Write a query** to find all points of interest that are contained in at least one region (use ST_Contains)

2. **Write a query** to find which region contains each point of interest (show POI name and region name)

3. **Write a query** to count how many POIs are in each region

---

## Question 4: Buffer and Proximity Queries

### Given:

```sql
CREATE TABLE Roads (
    RoadID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    RoadType VARCHAR(50),
    Geometry GEOMETRY(LINESTRING, 2180) -- Projected CRS
);

CREATE TABLE StreetLamps (
    LampID SERIAL PRIMARY KEY,
    Location GEOMETRY(POINT, 2180)
);
```

### Tasks:

1. **Write a query** to count all street lamps within 10 meters of roads named 'Main Street'
   - Use ST_Buffer() with appropriate endcap parameter

2. **Write a query** to find all roads that pass within 50 meters of a specific point (20.5, 51.0)

3. **Explain** what endcap='flat' does in ST_Buffer() and why it's useful

---

## Question 5: Length Calculations with Intersection

### Given:

```sql
CREATE TABLE CityBoundary (
    CityID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Boundary GEOMETRY(POLYGON, 2180)
);

CREATE TABLE Highways (
    HighwayID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    RoadType VARCHAR(50), -- 'motorway', 'primary', 'secondary'
    Geometry GEOMETRY(MULTILINESTRING, 2180)
);
```

### Tasks:

1. **Write a query** to find the total length (in kilometers) of all motorways (RoadType='motorway') that lie within city boundaries

2. **Write a query** to find which city has the longest total highway network (all road types combined)

3. **Explain** why you need ST_Intersection() for this calculation

---

## Question 6: GEOMETRY vs GEOGRAPHY

### Scenario:

You have a global dataset of airports and need to:
- Calculate distances between airports worldwide
- Find airports within a certain radius of a point
- Calculate flight routes (great circle distances)

### Questions:

1. **Should you use GEOMETRY or GEOGRAPHY?** Explain why.

2. **Write CREATE TABLE** statement for an Airports table with both GEOMETRY and GEOGRAPHY columns

3. **Write a query** to find all airports within 500km of a given point using GEOGRAPHY

4. **List 3 differences** between GEOMETRY and GEOGRAPHY data types

---

## Question 7: Spatial Queries with Multiple Conditions

### Given:

```sql
CREATE TABLE Buildings (
    BuildingID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Type VARCHAR(50), -- 'residential', 'commercial', 'industrial'
    Height NUMERIC(5,2), -- in meters
    Footprint GEOMETRY(POLYGON, 2180),
    Entrance GEOMETRY(POINT, 2180)
);

CREATE TABLE ParkingLots (
    ParkingID SERIAL PRIMARY KEY,
    Capacity INTEGER,
    Location GEOMETRY(POLYGON, 2180)
);
```

### Tasks:

1. **Write a query** to find all commercial buildings taller than 20m that have at least one parking lot within 100m of their entrance

2. **Write a query** to find the total parking capacity available within 200m of each building (show building name and total capacity)

3. **Write a query** to find buildings whose footprint intersects with any parking lot (potential conflicts)

---

## Question 8: Coordinate System Transformations

### Given:

```sql
-- Data in WGS-84 (EPSG:4326)
CREATE TABLE GPSPoints (
    PointID SERIAL PRIMARY KEY,
    Name VARCHAR(100),
    Location GEOMETRY(POINT, 4326) -- longitude, latitude
);
```

### Tasks:

1. **Write a query** to transform all points from EPSG:4326 to EPSG:2180 (Poland)

2. **Write a query** to calculate distances between consecutive points after transformation

3. **Explain** when you need to transform coordinates and why

4. **List** at least 3 EPSG codes and what they represent

---

## Question 9: Spatial Aggregation

### Given:

```sql
CREATE TABLE Sensors (
    SensorID SERIAL PRIMARY KEY,
    SensorType VARCHAR(50),
    Location GEOMETRY(POINT, 2180),
    LastReading NUMERIC(8,2),
    Timestamp TIMESTAMP
);

CREATE TABLE MonitoringZones (
    ZoneID SERIAL PRIMARY KEY,
    ZoneName VARCHAR(100),
    Boundary GEOMETRY(POLYGON, 2180)
);
```

### Tasks:

1. **Write a query** to find the average sensor reading for each monitoring zone

2. **Write a query** to find zones that have no sensors inside them

3. **Write a query** to find the sensor with the highest reading within 1km of the center of each zone (use ST_Centroid)

---

## Question 10: Complex Spatial Analysis

### Scenario (Real-world):

You have:
- A city boundary (POLYGON)
- Green spaces/parks (MULTIPOLYGON)
- Bus stops (POINT)
- Bus routes (LINESTRING)

### Tasks:

1. **Write a query** to calculate the percentage of the city area covered by green spaces

2. **Write a query** to find all bus stops that are more than 500m away from any green space

3. **Write a query** to find bus routes that pass through at least 3 different green spaces

4. **Write a query** to find the largest green space that has no bus stop within 200m

---

# Tips for Lab 3 Questions:

## Common PostGIS Functions:

### Distance Functions:
- `ST_Distance(geom1, geom2)` - Returns distance (units depend on SRID)
- `ST_DistanceSpheroid(geom1, geom2, spheroid)` - Great circle distance
- `ST_DistanceSphere(geom1, geom2)` - Spherical distance (meters)
- `ST_DWithin(geom1, geom2, distance)` - True if within distance

### Area/Length Functions:
- `ST_Area(geometry)` - Returns area (units depend on SRID)
- `ST_Length(geometry)` - Returns length
- **Always use projected CRS (like EPSG:2180) for area/length in meters**

### Spatial Relationships:
- `ST_Contains(geom1, geom2)` - True if geom1 contains geom2
- `ST_Within(geom1, geom2)` - True if geom1 is within geom2
- `ST_Intersects(geom1, geom2)` - True if geometries intersect
- `ST_Overlaps(geom1, geom2)` - True if geometries overlap

### Geometric Operations:
- `ST_Buffer(geometry, distance)` - Creates buffer around geometry
- `ST_Intersection(geom1, geom2)` - Returns intersection geometry
- `ST_Union(geometry)` - Aggregate function to merge geometries
- `ST_Centroid(geometry)` - Returns center point

### Coordinate Transformation:
- `ST_Transform(geometry, target_srid)` - Transform to different CRS
- `ST_SetSRID(geometry, srid)` - Set SRID (doesn't transform!)
- `ST_SRID(geometry)` - Get SRID of geometry

### Constructors:
- `ST_MakePoint(lon, lat)` - Create point from coordinates
- `ST_GeomFromText('WKT', srid)` - Create from WKT string
- `ST_GeomFromGeoJSON(json)` - Create from GeoJSON

## Important EPSG Codes:

| EPSG | Description | Use Case |
|------|-------------|----------|
| 4326 | WGS-84 (lat/lon) | GPS, global data, web maps |
| 2180 | Poland (meters) | Area/distance calculations in Poland |
| 3857 | Web Mercator | Web mapping (Google Maps, OpenStreetMap) |
| 4269 | NAD83 | North America |

## GEOMETRY vs GEOGRAPHY:

| Feature | GEOMETRY | GEOGRAPHY |
|---------|----------|-----------|
| **Model** | Flat plane | Earth's ellipsoid |
| **Distance units** | Depends on SRID | Always meters |
| **Accuracy** | Good locally | Good globally |
| **Performance** | Faster | Slower |
| **When to use** | Local/regional data with projection | Global data, no projection needed |

## Common Patterns:

### Calculate area in km²:
```sql
SELECT
    name,
    ST_Area(ST_Transform(boundary, 2180)) / 1000000 AS area_km2
FROM regions;
```

### Find points within polygon:
```sql
SELECT p.name
FROM points p
JOIN regions r ON ST_Contains(r.boundary, p.location)
WHERE r.name = 'Region1';
```

### Count features within buffer:
```sql
SELECT COUNT(*)
FROM lamps l
WHERE ST_DWithin(
    l.location,
    (SELECT geometry FROM roads WHERE name = 'Main St'),
    10
);
```

### Transform and calculate distance:
```sql
SELECT
    ST_Distance(
        ST_Transform(city1.location, 2180),
        ST_Transform(city2.location, 2180)
    ) / 1000 AS distance_km
FROM cities city1, cities city2
WHERE city1.name = 'Warsaw' AND city2.name = 'Krakow';
```

## Test Writing Tips:

1. ✅ **Always specify SRID** when creating geometries
2. ✅ **Use ST_Transform** for area/length calculations (to projected CRS)
3. ✅ **Divide by 1000000** to convert m² to km²
4. ✅ **Divide by 1000** to convert m to km
5. ✅ **Use ST_DWithin** instead of ST_Distance when checking proximity (faster!)
6. ✅ **Remember coordinate order**: longitude first, then latitude in WGS-84
7. ✅ **Use appropriate endcap** in ST_Buffer ('flat' to avoid overlaps)
