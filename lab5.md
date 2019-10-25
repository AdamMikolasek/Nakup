# Lab 5 - R-Trees indices, Array and JSON datatypes

## 1.Write a query which finds all restaurants (point, amenity = 'restaurant') within 1000 meters of 'Fakulta informatiky a informačných technológií STU' (polygon). Select the restaurant name and distance in meters. Sort the output by distance - closest restaurant first.

*Query:*
```sql
with fiit as(
select * from planet_osm_polygon where name like 'Fakulta informatiky%')
select p.name,st_distance(f.way::geography,p.way::geography) as distance from planet_osm_point p
cross join fiit f
where distance  < '1000'	
and p.amenity = 'restaurant'
order by distance 
 ```

*Answer:*
"Bastion - Slovenská Koliba"	"35.742375541"

"Drag"	"366.312228092"

"Idyla"	"414.046695922"

"Družba"	"622.942884737"

"Venza (študentská jedáleň)"	"674.141796757"

"Kamel"	"749.299621785"

"Eat and Meet (študentská jedáleň)"	"771.717881845"

"Riviera"	"825.566535564"

"Seoul garden"	"833.397496062"



## 2. Check the query plan and measure how long the query takes. Now make it as fast as possible. Make sure to also use geo-indices, but don't expect large improvements. The dataset is small, and filtering in amenity='restaurant' will greatly limit the search space anyway.

Total query runtime: priblizne 200ms bez indexov.

Vytvorenie indexov:
```sql
create index point_index on planet_osm_point using gist ((way::geography))
create index polygon_index on planet_osm_polygon using gist ((way::geography))
create index name_index on planet_osm_polygon(name)
create index amenity_index on planet_osm_point(amenity)
```

Total querez runtime: priblizne 180ms. Zrýchlenie je, avšak nie je vôbec výrazné.

## 3. Update the query to generate a geojson. The output should be a single row containg a json array like this:
Query:
```sql
with fiit as(
select * from planet_osm_polygon where name like 'Fakulta informatiky%'
)
select array_to_json(array_agg(row_to_json(x))) as geojson from (
select p.name,st_distance(f.way::geography,p.way::geography) as distance from planet_osm_point p
cross join fiit f
where st_distance(f.way::geography,p.way::geography)  < '1000'	
and p.amenity = 'restaurant'
order by distance) x 
```
Answer:
```json
"[{"name":"Bastion - Slovenská Koliba","distance":35.742375541},{"name":"Drag","distance":366.312228092},{"name":"Idyla","distance":414.046695922},{"name":"Družba","distance":622.942884737},{"name":"Venza (študentská jedáleň)","distance":674.141796757},{"name":"Kamel","distance":749.299621785},{"name":"Eat and Meet (študentská jedáleň)","distance":771.717881845},{"name":"Riviera","distance":825.566535564},{"name":"Seoul garden","distance":833.397496062}]"
```