# Lab 9 - Elasticsearch

## Start by writing the SQL query or queries which will select the data that you want to import to Elasticsearch. 

sql```
with ulice as (
	select * from planet_osm_line p1
	where p1.name is not null   
	and p1.z_order is not null  
	and p1.z_order != 0 
),
bod as (
	select ST_GeomFromText('POINT(17.052430 48.153933)',4326)::geography as bod
	)
select p.name as restaurant, array_agg(u.name) as streets, count(u.name) as count from planet_osm_point p	
join ulice u
on st_dwithin(ST_Transform(p.way,4326)::geography,ST_Transform(u.way,4326)::geography,50)
join bod b
on st_dwithin(ST_Transform(p.way,4326)::geography,b.bod,1000)
where p.name is not null
and p.amenity = 'restaurant'
and u.name is not null
group by p.name
```

## Once you have the data, design the document

```
{
  "restaurant": "rePUBlike",
  "streets": ["Čavojského","Námestie sv. Františka"],
  "count": 2
}
```

## Prepare a mapping, feel free to do it "by hand", e.g. via Postman
```
PUT 192.168.99.100:9200/lab9
{
  "mappings": {
    "properties": {
      "restaurant": {
        "type": "text"
      },
      "streets": {
        "type": "text"
      },
      "count": {
        "type": "long"
      }
    }
  }
}
```

## Write a code in a language of your choice which will run the SQL queries
```
import json
import requests
import psycopg2

QUERY= """
    with ulice as (
	select * from planet_osm_line p1
	where p1.name is not null   
	and p1.z_order is not null  
	and p1.z_order != 0 
),
bod as (
	select ST_GeomFromText('POINT(17.052430 48.153933)',4326)::geography as bod
	)
select p.name as restaurant, array_agg(u.name) as streets, count(u.name) as count from planet_osm_point p	
join ulice u
on st_dwithin(ST_Transform(p.way,4326)::geography,ST_Transform(u.way,4326)::geography,50)
join bod b
on st_dwithin(ST_Transform(p.way,4326)::geography,b.bod,1000)
where p.name is not null
and p.amenity = 'restaurant'
and u.name is not null
group by p.name
"""

connection = psycopg2.connect(dbname='gis', host='localhost', port=5432, user='postgres')
cursor = connection.cursor()
cursor.execute(QUERY)
results = cursor.fetchall()

for row in results:  
        result = {
            "restaurant": row[0],
            "streets" : row[1],
            "count": row[2]
        }
        print(result);
        requests.post('http://192.168.99.100:9200/lab9/_doc', json=result)  
       

connection.close()
```



You can always check the data in elasticsearch by running a simple GET request at the _search endpoint.

```
{
    "took": 77,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 10,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "ECvNCm8Bu3t5BSkmLAZG",
                "_score": 1.0,
                "_source": {
                    "restaurant": "Rotunda",
                    "streets": [
                        "Kolískova",
                        "Kolískova"
                    ],
                    "count": 2
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "ESvNCm8Bu3t5BSkmLAZW",
                "_score": 1.0,
                "_source": {
                    "restaurant": "Amélia Restaurant",
                    "streets": [
                        "Silvánska",
                        "Levárska",
                        "Levárska",
                        "Levárska"
                    ],
                    "count": 4
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "EivNCm8Bu3t5BSkmLAZe",
                "_score": 1.0,
                "_source": {
                    "restaurant": "Kamel",
                    "streets": [
                        "Molecova",
                        "Molecova",
                        "Karloveská",
                        "Molecova",
                        "Karloveská",
                        "Karloveská",
                        "Karloveská"
                    ],
                    "count": 7
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "EyvNCm8Bu3t5BSkmLAZn",
                "_score": 1.0,
                "_source": {
                    "restaurant": "Reštika",
                    "streets": [
                        "Kresánkova",
                        "Cikkerova"
                    ],
                    "count": 2
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "FCvNCm8Bu3t5BSkmLAaI",
                "_score": 1.0,
                "_source": {
                    "restaurant": "U Smädného Mnícha",
                    "streets": [
                        "Majerníkova",
                        "Ľudovíta Fullu"
                    ],
                    "count": 2
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "FSvNCm8Bu3t5BSkmLAab",
                "_score": 1.0,
                "_source": {
                    "restaurant": "Mexiko",
                    "streets": [
                        "Pribišova"
                    ],
                    "count": 1
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "FivNCm8Bu3t5BSkmLAa9",
                "_score": 1.0,
                "_source": {
                    "restaurant": "Jedna báseň",
                    "streets": [
                        "Pribišova"
                    ],
                    "count": 1
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "FyvNCm8Bu3t5BSkmLAbI",
                "_score": 1.0,
                "_source": {
                    "restaurant": "rePUBlike",
                    "streets": [
                        "Čavojského",
                        "Námestie sv. Františka",
                        "Námestie sv. Františka"
                    ],
                    "count": 3
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "GCvNCm8Bu3t5BSkmLAbY",
                "_score": 1.0,
                "_source": {
                    "restaurant": "CREPES KISS Palacinky",
                    "streets": [
                        "Pernecká",
                        "Námestie sv. Františka",
                        "Námestie sv. Františka",
                        "Špieszova",
                        "Pernecká",
                        "Pernecká"
                    ],
                    "count": 6
                }
            },
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "GSvNCm8Bu3t5BSkmLAbi",
                "_score": 1.0,
                "_source": {
                    "restaurant": "Palacinkáreň Vikaro",
                    "streets": [
                        "Kresánkova",
                        "Kresánkova"
                    ],
                    "count": 2
                }
            }
        ]
    }
}
```

## Once you have the data, write the Elasticsearch query

```
{
	"query" : {
		"match" : {
			"restaurant" : "Kamel"
		}
	},
	"aggs" : {
		"all_views" : {
			"sum" : {
				"field" : "count"
			}
		}
	}
}

```

## Output
```
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 2.3960416,
        "hits": [
            {
                "_index": "lab9",
                "_type": "_doc",
                "_id": "EivNCm8Bu3t5BSkmLAZe",
                "_score": 2.3960416,
                "_source": {
                    "restaurant": "Kamel",
                    "streets": [
                        "Molecova",
                        "Molecova",
                        "Karloveská",
                        "Molecova",
                        "Karloveská",
                        "Karloveská",
                        "Karloveská"
                    ],
                    "count": 7
                }
            }
        ]
    },
    "aggregations": {
        "all_views": {
            "value": 7.0
        }
    }
}
```