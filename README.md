# Домашнее задание к занятию 12.5 "Реляционные базы данных: Индексы"

Домашнее задание нужно выполнить на датасете из презентации.

*Решение нужно прислать одним SQL файлом, содержащим запросы по всем заданиям.*

Любые вопросы по решению задач задавайте в чате учебной группы.

---

Задание можно выполнить как в любом IDE, так и в командной строке.

### Задание 1.

Напишите запрос к учебной базе данных, который вернет процентное отношение общего размера всех индексов к общему размеру всех таблиц.
```sql
SELECT  SUM(tt.DATA_LENGTH) , SUM(tt.INDEX_LENGTH),
CONCAT(  ROUND ((SUM(tt.INDEX_LENGTH) / SUM(tt.DATA_LENGTH)) *100), ' %')  AS Отношение
FROM INFORMATION_SCHEMA.TABLES tt
WHERE  tt.TABLE_SCHEMA = 'sakila' 
;
```
### Задание 2.

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места, Использование Distinct, агрегация с группировкой по не относящемся полям c.customer_id, f.title которые не имеют ни какаой логики
- оптимизируйте запрос 

```sql
explain analyze
select distinct concat(c.last_name, ' ', c.first_name),
sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and 
r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
;

-> Limit: 20000 row(s)  (cost=0.00..0.00 rows=0) (actual time=6338.643..6338.735 rows=391 loops=1)
    -> Table scan on <temporary>  (cost=2.50..2.50 rows=0) (actual time=6338.640..6338.706 rows=391 loops=1)
        -> Temporary table with deduplication  (cost=0.00..0.00 rows=0) (actual time=6338.637..6338.637 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=2908.550..6118.442 rows=642000 loops=1)
                -> Sort: c.customer_id, f.title  (actual time=2908.495..2997.623 rows=642000 loops=1)
                    -> Stream results  (cost=22211557.90 rows=16700349) (actual time=0.421..2144.208 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=22211557.90 rows=16700349) (actual time=0.415..1860.647 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=20537347.88 rows=16700349) (actual time=0.411..1635.544 rows=642000 loops=1)
                                -> Nested loop inner join  (cost=18863137.85 rows=16700349) (actual time=0.404..1361.625 rows=642000 loops=1)
                                    -> Inner hash join (no condition)  (cost=1608774.80 rows=16086000) (actual time=0.391..65.670 rows=634000 loops=1)
                                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.68 rows=16086) (actual time=0.034..6.962 rows=634 loops=1)
                                            -> Table scan on p  (cost=1.68 rows=16086) (actual time=0.021..4.859 rows=16044 loops=1)
                                        -> Hash
                                            -> Covering index scan on f using idx_title  (cost=103.00 rows=1000) (actual time=0.049..0.262 rows=1000 loops=1)
                                    -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.97 rows=1) (actual time=0.001..0.002 rows=1 loops=634000)
                                -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)
                            -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)



explain analyze
select  concat(c.last_name, ' ', c.first_name) as pio,
sum(p.amount)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and 
r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
group by pio
;


-> Limit: 20000 row(s)  (actual time=3178.101..3178.230 rows=391 loops=1)
    -> Table scan on <temporary>  (actual time=3178.099..3178.202 rows=391 loops=1)
        -> Aggregate using temporary table  (actual time=3178.094..3178.094 rows=391 loops=1)
            -> Nested loop inner join  (cost=22211510.36 rows=16700349) (actual time=0.346..1878.471 rows=642000 loops=1)
                -> Nested loop inner join  (cost=20537300.33 rows=16700349) (actual time=0.342..1639.019 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=18863090.30 rows=16700349) (actual time=0.334..1360.363 rows=642000 loops=1)
                        -> Inner hash join (no condition)  (cost=1608727.25 rows=16086000) (actual time=0.322..52.244 rows=634000 loops=1)
                            -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.63 rows=16086) (actual time=0.036..6.607 rows=634 loops=1)
                                -> Table scan on p  (cost=1.63 rows=16086) (actual time=0.024..4.688 rows=16044 loops=1)
                            -> Hash
                                -> Covering index scan on f using idx_fk_language_id  (cost=103.00 rows=1000) (actual time=0.037..0.206 rows=1000 loops=1)
                        -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.97 rows=1) (actual time=0.001..0.002 rows=1 loops=634000)
                    -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)
                -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)

                
explain analyze
select  concat(c.last_name, ' ', c.first_name) as pio,
sum(p.amount)
from payment p, rental r, customer c, inventory i
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and 
r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
group by pio
;                

-> Limit: 20000 row(s)  (actual time=9.330..9.419 rows=391 loops=1)
    -> Table scan on <temporary>  (actual time=9.328..9.391 rows=391 loops=1)
        -> Aggregate using temporary table  (actual time=9.326..9.326 rows=391 loops=1)
            -> Nested loop inner join  (cost=30577.46 rows=16700) (actual time=0.076..8.603 rows=642 loops=1)
                -> Nested loop inner join  (cost=24732.34 rows=16700) (actual time=0.073..7.794 rows=642 loops=1)
                    -> Nested loop inner join  (cost=18887.21 rows=16700) (actual time=0.065..7.097 rows=642 loops=1)
                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1632.85 rows=16086) (actual time=0.051..5.628 rows=634 loops=1)
                            -> Table scan on p  (cost=1632.85 rows=16086) (actual time=0.039..4.176 rows=16044 loops=1)
                        -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.97 rows=1) (actual time=0.001..0.002 rows=1 loops=634)
                    -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=642)
                -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=642)
```

(внесите корректировки по использованию операторов, при необходимости добавьте индексы). 

Использование агрегатной функции с группировкой по имени_фамилии 
отказ от использования таблицы film f. Делает использование запроса ликвидным

## Дополнительные задания (со звездочкой*)
Эти задания дополнительные (не обязательные к выполнению) и никак не повлияют на получение вами зачета по этому домашнему заданию. Вы можете их выполнить, если хотите глубже и/или шире разобраться в материале.

### Задание 3*.

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL нет.

*Приведите ответ в свободной форме.*
