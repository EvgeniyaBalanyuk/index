# Домашнее задание к занятию «Индексы» - Баланюк Евгения

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### ОТВЕТ:
```
SELECT table_schema, CONCAT(ROUND((SUM(index_length)) * 100 / (SUM(data_length + index_length)), 2), '%') 'index'
FROM information_schema.TABLES where TABLE_SCHEMA = 'sakila';
```

![](https://github.com/EvgeniyaBalanyuk/index/blob/main/index.png)

---

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### ОТВЕТ:

В получении результата участвуют 2 таблицы: payment и customer

```
SELECT DISTINCT concat(c.last_name, ' ', c.first_name), sum(p.amount)
FROM payment p 
INNER JOIN customer c ON c.customer_id = p.customer_id 
WHERE p.payment_date >= '2005-07-30' and p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY c.customer_id; 
```

Анализ исходного запроса
```
(cost=2.5..2.5 rows=0) (actual time=5007..5007 rows=391 loops=1)
```

Анализ оптимизированного запроса

```
-> Sort with duplicate removal: `concat(c.last_name, ' ', c.first_name)`, `sum(p.amount)`  (actual time=2.55..2.58 rows=391 loops=1)
    -> Table scan on <temporary>  (actual time=2.3..2.35 rows=391 loops=1)
        -> Aggregate using temporary table  (actual time=2.3..2.3 rows=391 loops=1)
            -> Nested loop inner join  (cost=507 rows=634) (actual time=0.0242..1.94 rows=634 loops=1)
                -> Index range scan on p using pay_date over ('2005-07-30 00:00:00' <= payment_date < '2005-07-31 00:00:00'), with index condition: ((p.payment_date >= TIMESTAMP'2005-07-30 00:00:00') and (p.payment_date < <cache>(('2005-07-30' + interval 1 day))))  (cost=286 rows=634) (actual time=0.0169..0.982 rows=634 loops=1)
                -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=0.00126..0.0013 rows=1 loops=634)

```

---
