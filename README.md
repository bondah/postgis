## Follow along at: [https://bondah.github.io/postgis](https://bondah.github.io/postgis)

# Objective
In this tutorial we will learn:
* how to add data to a postgreSQL database
* how to perform a SQL query on the data
* how to view and edit the data in QGIS

# What data will we use?
We are using GIS data from (exercise data is in the data folder):
* LA County: [https://data.lacounty.gov/GIS-Data/Streams-and-Rivers/6bsh-b6vg](LA County Rivers and Streams) (this has been edited slightly to simplify this exercise. Unnamed streams were removed and all rivers were dissolved by name)
* 2010 Census Tracts: [http://www2.census.gov/geo/tiger/TIGER2010DP1/Tract_2010Census_DP1.zip](2010 Census tracts w/ data) (LA County has been extracted for this exercise)

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
