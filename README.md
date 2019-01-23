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
![Image alt](https://github.com/Denisplusplus/something/blob/master/images/schema_model.png)
* Используем СУБД: PostgreSQL + PgAdmin
![Image alt](https://github.com/Denisplusplus/something/blob/master/images/postgres_schema.png)
* Тестовые данные:
![Image alt](https://github.com/Denisplusplus/something/blob/master/images/test_data.png)
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
![Image alt](https://github.com/Denisplusplus/something/blob/master/images/query_result.png)



### Вопрос №2

С сайта google news (https://news.google.com) (язык и регион - English | United States) необходимо прокачать все статьи за последний месяц (на момент прокачки) с ключевым словом Russia.

Затем для скачанных статей необходимо рассчитать топ-50 наиболее частотных слов и представить их в виде word (tag) cloud. Данное задание необходимо выполнить с помощью python. Для представления в виде word cloud можно использовать уже существующие библиотеки. Пример word cloud можно посмотреть по ссылке - https://altoona.psu.edu/sites/altoona/files/styles/photo_gallery_large/http/news.psu.edu/sites/default/files/success-word-cloud.jpg

### Решение:
