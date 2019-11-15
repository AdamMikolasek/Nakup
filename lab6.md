# Lab 6: Recursive SQL

## 1. Write a recursive query which returns a number and its factorial for all numbers up to 10
```sql
with recursive fact(n, fact) as (
select 0, 1
union all
select (f.n+1), f.fact * (f.n+1) from fact as f
)
select * from fact 
limit 10
 ```

## 2.Write a recursive query which returns a number and the number in Fibonacci sequence at that position for the first 20 Fibonacci numbers.
```sql
with recursive fib(n,prev,fib) as (
select 1,1,1
union all	
select f.n+1, f.fib, f.fib + f.prev from fib as f
)
select n,fib from fib
limit 20
```

## 3. Table product_parts contains products and product parts which are needed to build them. A product part may be used to assemble another product part or product, this is stored in the part_of_id column. When this column contains NULL it means that it is the final product. List all parts and their components that are needed to build a 'chair'.

```sql
with recursive chair(part,id,part_of_id) as (
select p.name,p.id,p.part_of_id from product_parts as p
where name = 'chair'
union all
select p.name, p.id,p.part_of_id from chair as c
join product_parts as p	on p.part_of_id = c.id
)
select part from chair
where part_of_id is not null
```
## 4. List all bus stops between 'Zochova' and 'Zoo' for line 39. Also include the hop number on that trip between the two stops. 
```sql
with recursive zastavky(name,start_stop_id,end_stop_id,duration) as (
select  s.name,c.start_stop_id, c.end_stop_id, c.duration from connections c
join stops s on c.start_stop_id = s.id
where c.line = '39' and s.name = 'Zoo'

union

select s.name, c.start_stop_id, c.end_stop_id,c.duration from zastavky z
join connections c on z.end_stop_id = c.start_stop_id
join stops s on s.id = z.end_stop_id
where c.line = '39' and s.name != 'Zochova'
)
select * from zastavky
where name != 'Zoo'
```
## Optional 1. Which one of all the parts that are used to build a 'chair' has longest shipping time?

```sql
with recursive chair(part,id,part_of_id,shipping_time) as (
select p.name,p.id,p.part_of_id,p.shipping_time from product_parts as p
where name = 'chair'
union all
select p.name, p.id,p.part_of_id,p.shipping_time from chair as c
join product_parts as p	on p.part_of_id = c.id
)
select part,shipping_time from chair
where part_of_id is not null
order by  shipping_time DESC
limit 1
```

