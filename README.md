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
```sql
SELECT rivers.gnis_name as rivers, count(*) as tracts
	FROM rivers, tracts
	WHERE ST_Intersects(rivers.geom, tracts.geom)
	GROUP BY rivers
    ORDER BY rivers;
```

# Advanced
