# 12_05 Стасенко Григорий Домашнее задание к занятию «Индексы» 12_05

## Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

![image](https://github.com/Nightnek/HW12-5/assets/127677631/4074bc4a-3ddc-4f78-8cc1-3398f9c833e7)

````
SELECT SUM(DATA_LENGTH), SUM(INDEX_LENGTH),
		SUM(INDEX_LENGTH)/SUM(DATA_LENGTH+INDEX_LENGTH) *100 AS '% of index'
From information_schema.tables
WHERE TABLE_SCHEMA = 'sakila';
````
---

## Задание 2
Выполните explain analyze следующего запроса:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
перечислите узкие места;
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

````
В данном запросе таблица film не используется, соответственно, нет необходимости подключать и теблицу inventory.
А так как цель запроса- вывод платежей за определенную дату, нам и таблица rental в данном запросе не нужна.
А информацию о клиенте мы можем не тянуть через таблицу связку таблиц paymant - rental - customer, связав срезу таблицы payment и customer.
Таким образом, все еобходимые данные есть в таблицах payment и customer.
Убрав лишнее, мы получаем следующий запрос:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id

Результат тот же, а время обработки запроса уменьшилось в 909 раз, так как мы убрали соединения с ненужными в данном запросе таблицами
До -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=11827..11827 rows=391 loops=1)
После -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=13..13.1 rows=391 loops=1)
````
---

