# Súbor obsahujúci query použité v projekte

## Query na vyhľadanie podnikov v mestskej časti.
Toto query slúži na vyhľadanie všetkých krčiem, reštaurácií a kaviarní v mestskej časti. Mestská časť je v programe zadaná ako argument od používateľa, tu je skopírované pre čast `Bratislava - mestská čast Staré Mesto`.


```sql
with mestska_cast as (
select * from planet_osm_polygon
where name = 'Bratislava - mestská časť Staré Mesto'
and admin_level = '9')
select p1.name, p1.amenity, ST_AsGeoJSON(ST_transform(p1.way,4326)) from planet_osm_point p1
cross join mestska_cast mc
where p1.name is not null
and (p1.amenity = 'pub' or p1.amenity = 'restaurant' or p1.amenity = 'cafe')
and st_contains(mc.way,p1.way)
limit 10
```

## Query na vyhľadanie najbližšej ulice k používaťeľovej pozícii. 
V tomto query je pozícia používateľa nahradená fixným bodom. Toto query slúži ako vstup do rekurzívneho query, kde je potrebné poznať ID ulice na ktorej sa používateľ nachádza (alebo je k nej najbližšie).
```sql
select p1.osm_id, p1.name,
ST_distance(ST_GeomFromText('POINT(17.064271599999998 48.15826)',4326)::geography, ST_Transform(p1.way,4326)::geography)
from planet_osm_line p1   
where p1.name is not null  
and p1.z_order is not null   
and p1.z_order != 0  
and ST_Dwithin(ST_GeomFromText('POINT(17.064271599999998 48.15826)',4326)::geography, ST_Transform(p1.way,4326)::geography,100) 
order by ST_distance(ST_GeomFromText('POINT(17.064271599999998 48.15826)',4326)::geography, ST_Transform(p1.way,4326)::geography) asc  
limit 1
```

## Query na vyhľadanie najbližšej ulice k podniku, do ktorého chce používateľ navigovať. 
V tomto query je fixne nastavený podnik `Zbrojnoš`, ktorý v aplikácii môže samozrejme používateľ meniť. Toto query slúži ako vstup do rekurzívneho query, kde je potrebné poznať ID ulice na ktorej sa podnik nachádza (alebo je k nej najbližšie).
```sql
with podnik as ( 
select osm_id, way from planet_osm_point 
where name like 'Zbrojnoš'  
) 
select p1.osm_id, p1.name, 
st_distance(ST_transform(p1.way,4236)::geography,ST_transform(p2.way,4236)::geography)
from planet_osm_line p1
cross join podnik p2 
where p1.name is not null  
and p1.z_order is not null
and p1.z_order != 0
and st_dwithin(ST_transform(p1.way,4236)::geography,ST_transform(p2.way,4236)::geography,100)  
order by st_distance(ST_transform(p1.way,4236)::geography,ST_transform(p2.way,4236)::geography) asc 
limit 1
```

## Veľké rekurzívne query na hľadanie cesty
Toto query slúži na hľadanie najkratšej cesty od miesta kde sa používateľ nachádza až k podniku kam sa chce dostať. Funkcionalita by sa dala opísať nasledovne: Zoberie sa ulica, pri ktorej sa používateľ nachádza a zistia sa všetky ulice, ktoré sa s ňou pretínajú a body týchto križovatiek. Následne sa podobne prehľadávajú susediace ulice a tak ďalej, až kým sa nepríde k ulici, na ktorej sa nachádza podnik. Cesta potom vznikne pospájaním týchto bodov križovatiek.

```sql
with recursive cesty(bod_pretnutia, druha_cesta, pole, mena, ids, druha_id,way,distance) as (			 
	select (st_intersection(p1.way,p2.way)), p2.name, 
	array[ST_AsGeoJSON(st_intersection(st_transform(p1.way,4326)::geography,st_transform(p2.way,4326)::geography))], 
	array[p1.name, p2.name], array[p1.osm_id,p2.osm_id],p2.osm_id, p2.way, '0'::DOUBLE PRECISION
	from planet_osm_line as p1
	join planet_osm_line as p2
	on ST_Intersects(p1.way,p2.way)
	where p1.name is not null and p2.name is not null 
	and p1.osm_id = '45458994' and p1.name != p2.name 
	and p1.z_order is not null and p2.z_order is not null 
	and p1.z_order != 0 and p2.z_order != 0
	
union all
	
	select (st_intersection(c.way,p4.way)), p4.name, 
	c.pole ||ST_AsGeoJSON(st_intersection(st_transform(c.way,4326)::geography,st_transform(p4.way,4326)::geography)),
	c.mena || p4.name, c.ids || p4.osm_id, p4.osm_id, p4.way, 
	(c.distance + st_distance(c.bod_pretnutia,(st_intersection(c.way,p4.way)))) 
	from cesty c 
	join planet_osm_line as p4 on  ST_Intersects(c.way,p4.way)
	where c.druha_id != 93210929  
	and p4.name is not null 
	and p4.osm_id != c.druha_id 
	and p4.z_order is not null 
	and p4.z_order != 0
	and array_length(c.pole, 1) <= 3
	
	),
start as (
	select ST_AsGeoJSON(ST_GeomFromText('POINT(17.052430 48.153933)',4326)::geography) as bod, 'start' as name
),
koniec as (
	select name,ST_AsGeoJSON(ST_transform(way,4326)::geography) as bod from planet_osm_point
	where name like 'Mexiko'
)
select  array['start'] || c.mena || e.name as name, array[s.bod] || c.pole || e.bod as points
from cesty c
cross join start s
cross join koniec e
where c.druha_id = 93210929 			 
order by distance asc
limit 1
```