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
import requests
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
* За 23.01.2019

![Image alt](https://github.com/Denisplusplus/something/blob/master/images/wordCloud23.png)

* За 22.01.2019

![Image alt](https://github.com/Denisplusplus/something/blob/master/images/wordCloud22.png)

* За период с 21.12.2018 по 21.01.2019

![Image alt](https://github.com/Denisplusplus/something/blob/master/images/wordCloud21-21.png)

### Вопрос №3

Оцените количество точек на картинке (Любым способом на ваш выбор) и опишите, как вы это сделали.
https://lh5.googleusercontent.com/yO12GARP3fqmNOZ00zM9Q_nyBVWWfR_xVu8skrvAmhB1hzSJyq_F593jhQqS48aWJyCZ5jzDAQ=w513


### Решение:

Среди придуманных вариантов решения хочу рассказать об этих:
* Разделить картинку на части(области) и посчитать количество точек попавших в одну область. Затем умножить на количество областей. 
Этот вариант подразумевает, что в каждую область в среднем попадет одно и то же число точек. Т.е. точки распределены равномерно по картинке. Смотря на картинку, мы видим, что распределение точек не равномерное, следовательно погрешеность будет существенной
* Посчитать количество пикселей, которые составляют нашу "точку". Затем посчитать количество белых пикселей. Далее из всего количества пикселей (которое мы можем легко узнать - ширина * высота в пикселях) вычесть число белых пикселей и разделить на количество пикселей, которые составляют "точку". Здесь так же видим ограничение. Оно состоит в том, что этот бы вариант хорошо работал, если бы не было наслаивания(пересечения) точек. Но, смотря на картинку, мы видим, что пересечения точек имеются
* Разобрать пиксельное содержимое одной точки, включая главное - цвет. Далее проходить в цикле по всем пикселям и сравнивать наш шаблон с пикселями на картинке. Данный вариант основывается на том, что пиксельное содержимое каждой точки на картинке одинаковое. 

Последний вариант показался мне наиболее правдоподобным, поэтому я реализовал его на Python

Исходный код:
```
from PIL import Image, ImageDraw 

image = Image.open("temp.png")
width = image.size[0] 
height = image.size[1] 

pix = image.load()


def checkMatch(i, j):
	if ((pix[i, j+2] != (255,255,255, 255)) and
		(pix[i, j+3] != (255,255,255, 255)) and
		(pix[i, j+4] != (255,255,255, 255)) and
		(pix[i, j+5] != (255,255,255, 255)) and
		(pix[i, j+6] != (255,255,255, 255)) and

		(pix[i+1, j+2] != (255,255,255, 255)) and
		(pix[i+1, j+3] != (255,255,255, 255)) and
		(pix[i+1, j+4] != (255,255,255, 255)) and
		(pix[i+1, j+5] != (255,255,255, 255)) and
		(pix[i+1, j+6] != (255,255,255, 255)) and
		(pix[i+1, j+7] != (255,255,255, 255)) and

		(pix[i+2, j+2] != (255,255,255, 255)) and
		(pix[i+2, j+3] != (255,255,255, 255)) and
		(pix[i+2, j+4] != (255,255,255, 255)) and
		(pix[i+2, j+5] != (255,255,255, 255)) and
		(pix[i+2, j+6] != (255,255,255, 255)) and
		(pix[i+2, j+7] != (255,255,255, 255)) and

		(pix[i+3, j+2] != (255,255,255, 255)) and
		(pix[i+3, j+3] != (255,255,255, 255)) and
		(pix[i+3, j+4] != (255,255,255, 255)) and
		(pix[i+3, j+5] != (255,255,255, 255)) and
		(pix[i+3, j+6] != (255,255,255, 255)) and
		(pix[i+3, j+7] != (255,255,255, 255)) and

		(pix[i+4, j+2] != (255,255,255, 255)) and
		(pix[i+4, j+3] != (255,255,255, 255)) and
		(pix[i+4, j+4] != (255,255,255, 255)) and
		(pix[i+4, j+5] != (255,255,255, 255)) and
		(pix[i+4, j+6] != (255,255,255, 255))):
			return True
	return False		


countDots = 0

for i in range(width - 4):
	for j in range(height - 7):
			if (checkMatch(i,j)):
				countDots+=1

print("Approximate dot's count ~=", countDots)
```

Результат оказался равным 841


