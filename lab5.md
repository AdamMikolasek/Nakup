#Lab 5 - R-Trees indices, Array and JSON datatypes

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
