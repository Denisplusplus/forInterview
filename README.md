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
* Для начала нам необходимо взять контент статей. Самый подходящий вариант - использовать API news.google.com
* Вторым шагом будет создание объекта, где ключ - это слово, значение - это количество повторений 
* Далее мы отберем топ 50 слов по встречаемости и воспользуемся библиотекой wordcloud, и получим готовое облако слов

Исходный код:
```
quests
import json 
import nltk
from nltk.corpus import stopwords 
from nltk.tokenize import word_tokenize 
from wordcloud import WordCloud
import matplotlib.pyplot as plt
import numpy as np

response = requests.get('https://newsapi.org/v2/everything?q=russia&pageSize=100&page=10&from=2019-01-23&to=2019-01-23&apiKey=KEY')
jsonText = json.loads(response.text)

text = ""

for obj in jsonText['articles']:
	text+=str(obj['content']).lower()

stopWords = set(stopwords.words('english')) 
wordTokens = word_tokenize(text)
filteredText = [word for word in wordTokens if not word in stopWords and word.isalpha() and word != 'chars'] 

dataFrequencies = {}

for word in filteredText:
    dataFrequencies[word] = dataFrequencies.get(word, 0) + 1 
  
dataFrequencies50 = dict(sorted(dataFrequencies.items(), key=lambda f:-f[1])[:50])

wordCloud = WordCloud(width=900,height=500, max_words=50, background_color="white", relative_scaling=1,normalize_plurals=False).generate_from_frequencies(dataFrequencies50)

plt.imshow(wordCloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```

--- 
В процессе решения столкнулся с проблемой наличия предлогов, служебных слов в тексте. Самым простым и, наверное, наиболее эффективным вариантом решения этой проблемы оказалось использование библиотеки nltk.

Неприятным моментом оказалось то, что нельзя получить полный текст статьи (При прохождении лимита приходит ответ, что необходимо платить, чтобы получить больше контента). Из-за этого встречаются [+n chars]. Принял решение убрать слово chars.

Облако строим с помощью библиотек wordcloud и matplotlib

### Результат:
За 23.01.2019
![Image alt](https://github.com/Denisplusplus/something/blob/master/images/query_result.png)
За 22.01.2019
![Image alt](https://github.com/Denisplusplus/something/blob/master/images/query_result.png)
За период с 21.12.2018 по 21.01.2019
![Image alt](https://github.com/Denisplusplus/something/blob/master/images/query_result.png)
