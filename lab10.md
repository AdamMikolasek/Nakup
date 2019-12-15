# Lab 10- Elasticsearch

## Check cluster nodes via `curl -s 'localhost:9200/_cat/nodes?v'`.

Všetky nodes sa pripojili, master ostal rovnaky aj pri opakovanom skúšaní (esmaster02).

## Shut down the current master.

```
docker stop esmaster02
```

Novým masterom sa stal esmaster03, `"status" : "green",`


## While you are at this stage, with 2 masters-eligible nodes, create an index with 2 shards and 1 replica and try indexing some data. 


```
PUT localhost:9200/lab

{
    "settings": {
        "index": {
            "number_of_shards": 2,
            "number_of_replicas": 1
        }
    }
}
```

## Stop another master-eligible node. Just to get a point across - try stopping the master-eligible node that is not currently elected master.

```
docker stop esmaster01
```
`localhost:9200/_cat/nodes?v` vratilo error


## In this degraded state, try searching for data. A simple search at the _search enpoint is enough, do not bother writing a query. Does searching still work?

Search stále funguje.

## Bring back one of the dead master-eligible nodes.
`docker start esmaster01`

Podarilo sa nastartovat bez problemov, nastartovanie zvysnej.

`docker start esmaster02`

## Inspect your shard layout via `curl localhost:9200/_cat/shards`.

Vybral som node es02, takže `docker stop es02`. Status ostal green, po minúte bol tiež green. 

## Take down another node and repeat tasks from 7.
`docker stop es01`

## Try searching and indexing in this state. Are both operations working?
Obe fungujú, avšak status sa zmenil z green na yellow.

## Take down the last data node. 
Po stopnutí posledného nodu sa zmenil status na red.

## Bring back the data nodes one by one and observe what happens.
Status postupne prešiel naspať na green, pričom na dátach nebola spozorovaná žiadna strata.


 

