## Follow along at: [https://bondah.github.io/postgis](https://bondah.github.io/postgis)

# Objective
In this tutorial we will learn:
* how to add data to a postgreSQL database
* how to perform a SQL query on the data
* how to view and edit the data in QGIS

# What data will we use?
We are using GIS data from (exercise data is in the data folder):
* LA County: [LA County Streams and Rivers](https://data.lacounty.gov/GIS-Data/Streams-and-Rivers/6bsh-b6vg) (this has been edited slightly to simplify this exercise. Unnamed streams were removed and all rivers were dissolved by name)
* 2010 Census Tracts: [2010 Census Tracts w/ data](http://www2.census.gov/geo/tiger/TIGER2010DP1/Tract_2010Census_DP1.zip (LA County has been extracted for this exercise)

# Scope
We aren't going to be using large datasets for efficiency, but the skills you learn can be applied to large datasets and complex queries.

# Code snippets
Intersection of tracts and rivers:
```sql
SELECT rivers.gnis_name as rivers, count(*) as tracts
	FROM rivers, tracts
	WHERE ST_Intersects(rivers.geom, tracts.geom)
	GROUP BY rivers
  ORDER BY rivers;
```
Create view of tracts and rivers intersection:
```sql
CREATE VIEW tracts_touch_rivers
	AS SELECT rivers.gnis_name as rivers, count(*) as tracts
		FROM rivers, tracts
		WHERE ST_Intersects(rivers.geom, tracts.geom)
		GROUP BY rivers
	  ORDER BY rivers;
```
Select tracts with more than 500 vacant units:
```sql
SELECT id, geom, geoid10, namelsad10, DP0180003  
	FROM tracts WHERE DP0180003 > 500;
```
Create a table of of tracts with more than 500 vacant units:
```sql
CREATE TABLE vacant_units AS
	SELECT id, geom, geoid10, namelsad10, DP0180003  
		FROM tracts WHERE DP0180003 > 500;
```
Create a buffer of the LA River:
```sql
CREATE TABLE lariver_mile_buffer AS
	SELECT ST_Buffer(geom, 5280) AS geom
		FROM rivers
		WHERE gnis_name = 'Los Angeles River';

ALTER TABLE lariver_mile_buffer ADD COLUMN gid serial;
ALTER TABLE lariver_mile_buffer ADD PRIMARY KEY (gid);
```

Create proportion function:
```sql
CREATE OR REPLACE FUNCTION proportional_sum(project geometry, base geometry, sumnum double precision)
RETURNS double precision AS
$BODY$

WITH myintersection AS
	(SELECT ST_Intersection(project, base) AS geom),
intarea AS
	(SELECT ST_Area(geom) AS intarea FROM myintersection),
sumarea AS
	(SELECT ST_Area(base) AS sumarea),
proportion AS
	(SELECT intarea / sumarea AS proportion FROM
		intarea CROSS JOIN sumarea)
SELECT sumnum * proportion FROM proportion;

$BODY$
LANGUAGE sql VOLATILE;
```

Use proportion function on our data:
```sql
WITH proportion AS
	(SELECT river.gid, proportional_sum(river.geom, tracts.geom, tracts.DP0010001) AS populations FROM
		lariver_mile_buffer AS river, tracts as tracts
	WHERE ST_Intersects(river.geom, tracts.geom))
SELECT ROUND(SUM(populations)) AS population FROM proportion
	GROUP BY gid;
```
