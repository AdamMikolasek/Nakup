# Benchmarking PostgreSQL write performance

## Vybrany jazyk
Script som pisal v pythone, pricom benchmark funkciu som si vytvoril sam
Benchmark funkcia mala na vstupe funkciu insert a pocet opakovani a vracala cas vykonania
Benchmark funkcia bola spustana pre dva rozne pocty - 10 000 a 20 000 opakovani

```
def timereps(reps, func):
    start = time()
    for i in range(0, reps):
        func()
    end = time()
    return (end - start)

```


```
def insert():
    time = datetime.datetime.now()
    INSERT_QUERY = "INSERT INTO documents (name,type,created_at,department, contracted_amount)" \
               "VALUES (%s, %s, %s, %s, %s)"

    QUERY_VALUES = ('Name', 'MyType', time, 'MyDep', '100')
    cursor.execute(INSERT_QUERY, QUERY_VALUES);
    conn.commit()
```