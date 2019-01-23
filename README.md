# Задание

### Вопрос №1
У вас SQL база с таблицами:
1) Users(userId, age)
2) Purchases (purchaseId, userId, itemId, date)
3) Items (itemId, price).

Напишите SQL запрос, который посчитает, сколько в среднем на покупки в месяц тратит:
- пользователь в возрастном диапазоне от 18 до 25 лет включительно
- пользователь в возрастном диапазоне от 26 до 35 лет включительно
- пользователь любого возраста

### Решение:
* Модель схемы можем представить следующим образом:
![Image alt](https://github.com/Denisplusplus/SSCC/raw/master/Others/1.png)
* Используем СУБД: PostgreSQL + PgAdmin
![Image alt](https://github.com/Denisplusplus/SSCC/raw/master/Others/1.png)
* Тестовые данные:
![Image alt](https://github.com/Denisplusplus/SSCC/raw/master/Others/1.png)
* Итоговый запрос выглядит следующим образом:
```
SELECT "PurchasesSchema"."Purchases".user_id                  AS USER, 
       "PurchasesSchema"."Users".age                          AS AGE,
       AVG("PurchasesSchema"."Items".price)                   AS PRICE,
       date_part('month', "PurchasesSchema"."Purchases".date) AS MONTH
FROM 

"PurchasesSchema"."Purchases" 
LEFT JOIN "PurchasesSchema"."Users" ON "PurchasesSchema"."Purchases".user_id = "PurchasesSchema"."Users".user_id 
LEFT JOIN "PurchasesSchema"."Items" ON "PurchasesSchema"."Purchases".item_id = "PurchasesSchema"."Items".item_id

WHERE "PurchasesSchema"."Users".age >= 18 AND "PurchasesSchema"."Users".age <= 25

GROUP BY date_part('month', "PurchasesSchema"."Purchases".date) , 
	"PurchasesSchema"."Purchases".user_id, 
	"PurchasesSchema"."Users".age
```
* Результат:
![Image alt](https://github.com/Denisplusplus/SSCC/raw/master/Others/1.png)
