# Домашнее задание к занятию «Индексы» - Баланюк Евгения

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### ОТВЕТ:
```
SELECT table_schema, CONCAT(ROUND((SUM(index_length)) * 100 / (SUM(data_length + index_length)), 2), '%') 'index'
FROM information_schema.TABLES where TABLE_SCHEMA = 'sakila';
```



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
SELECT DISTINCT concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
FROM payment p 
JOIN customer c ON c.customer_id = p.customer_id 
WHERE DATE(p.payment_date) = '2005-07-30';
```

Анализ исходного запроса
```
(cost=2.5..2.5 rows=0) (actual time=5007..5007 rows=391 loops=1)
```

Анализ оптимизированного запроса

```
(cost=2.5..2.5 rows=0) (actual time=6.86..6.91 rows=391 loops=1)
```

---
