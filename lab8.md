# Benchmarking PostgreSQL write performance
Vsetky testy prebiehali na laptope s Windowsom 10

## Vybrany jazyk
Script som pisal v pythone, pricom benchmark funkciu som si vytvoril sam
Benchmark funkcia mala na vstupe funkciu insert a pocet opakovani a vracala cas vykonania
Benchmark funkcia bola spustana pre dva rozne pocty - 10 000 a 20 000 opakovani

### Definovanie benchmark funkcie

```
def timereps(reps, func):
    start = time()
    for i in range(0, reps):
        func()
    end = time()
    return (end - start)

```

### Definovanie insert funkcie
```
def insert():
    time = datetime.datetime.now()
    INSERT_QUERY = "INSERT INTO documents (name,type,created_at,department, contracted_amount)" \
               "VALUES (%s, %s, %s, %s, %s)"

    QUERY_VALUES = ('Name', 'MyType', time, 'MyDep', '100')
    cursor.execute(INSERT_QUERY, QUERY_VALUES);
    conn.commit()
```

### Vytvorenie database connection a volanie funkcie
```
import psycopg2
import datetime
from time import time

conn = psycopg2.connect(host="192.168.99.100",database="oz", user="postgres", password="183795")
cursor = conn.cursor()

run_time = timereps(10000, insert)
print(run_time)

```

### import psycopg2
import datetime
from time import time

### Vysledky
```
Normal:
8.950443029403687s - 10 000
8.319307327270508s - 10 000
8.653979539871216s - 10 000

20.195600748062134s - 20 000
17.322676181793213s - 20 000
16.200615167617798s - 20 000


Disabled synchronous commit:
5.950071811676025s -10 000
6.142171859741211s -10 000
6.142171859741211s -10 000

16.006224870681763s - 20 000
12.831838130950928s - 20 000
12.941046476364136s - 20 000
```

Z vysledkov mozeme vidiet:
Pri synchronnom commitovani bola rychlost insertovania priblizne 1000 az 1200 zaznamov za sekundu.
Pri asynchronnom commitovani bola rychlost insertovania priblizne 1600 az 1900 zaznamov za sekundu.
Pozorujeme teda zrychlenie, ale ani zdaleka nie take ake bolo v uvedenom priklade v labe.
Toto moze byt sposobene tym ze to bolo spustane na Windowse.