# Домашнее задание к занятию "`Индексы`" - `Холопко Антон`




### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

## Решение

![Задание1Скриншот1](https://github.com/Easyjetz/12-05.md/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B51%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%821.png)


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

## Решение

- Выполните explain analyze следующего запроса:

![Задание2Скриншот1](https://github.com/Easyjetz/12-05.md/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B52%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%821.png)

- Узкие места:


1)Отсутствие индексов на ключевых полях:

*payment_date в таблице payment

*rental_date в таблице rental

*inventory_id в таблице rental

2)Использование DATE() в условии WHERE:

*Преобразование DATE(p.payment_date) не позволяет использовать индекс

*Лучше использовать диапазон дат

3)Старый синтаксис JOIN:

*Неявные соединения (через запятую) менее читаемы

*Лучше использовать явные JOIN

4) Оконная функция с избыточным партиционированием

*Партиционирование по f.title создает много мелких групп

*Для этого запроса достаточно партиционирования по c.customer_id

---

## Оптимизация

1. `Добавление индексов`
- Для фильтрации по дате платежа
   
```
CREATE INDEX idx_payment_date ON payment(payment_date);
```

- Для соединения по дате аренды

```
CREATE INDEX idx_rental_date ON rental(rental_date);
```
- Для соединения по inventory_id

```
CREATE INDEX idx_rental_inventory ON rental(inventory_id);
```


2. `Оптимизированный запрос`
   
```
EXPLAIN ANALYZE
SELECT 
    CONCAT(c.last_name, ' ', c.first_name) AS customer_name,
    SUM(p.amount) AS total_amount
FROM payment p
JOIN rental r ON r.rental_id = p.rental_id
JOIN customer c ON c.customer_id = p.customer_id
WHERE p.payment_date >= '2005-07-30' 
  AND p.payment_date < '2005-07-31'
GROUP BY c.customer_id;
```

![Задание2Скриншот2](https://github.com/Easyjetz/12-05.md/blob/main/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B52%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%822.png)

