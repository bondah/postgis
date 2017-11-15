## Follow along at: [https://maptimela.github.io/postgis](https://maptimela.github.io/postgis)

# Objective
In this tutorial we will learn:
* how to add data to a postgreSQL database
* how to perform a SQL query on the data
* how to view and edit the data in QGIS

# What data will we use?
We are using GIS data from LA County

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
	SELECT rivers.gnis_name as rivers, count(*) as tracts
		FROM rivers, tracts
		WHERE ST_Intersects(rivers.geom, tracts.geom)
		GROUP BY rivers
	    ORDER BY rivers;
```
Select tracts with more than 500 vacant units:
```sql
SELECT * FROM tracts WHERE DP0180003 > 500;
```
Create a table of of tracts with more than 500 vacant units:
```sql
CREATE TABLE vacant_units AS
	SELECT * FROM tracts WHERE DP0180003 > 500;
```
Create a buffer of the LA River:
```sql
CREATE TABLE lariver_mile_buffer AS
	SELECT ST_Buffer(geom, 5280) AS geom
		FROM rivers
		WHERE gnis_name = 'Los Angeles River';
```

# Advanced
