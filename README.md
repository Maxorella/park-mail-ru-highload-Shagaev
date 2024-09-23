# Проектирование высоконагруженных систем
## 1. Тема и целевая аудитория
### Тема: X

Информация о сервисе и нагрузке:
- DAU 217 миллионов пользователей. (в лекции было 436, взято из [2](https://www.meltwater.com/en/blog/twitter-stats-marketers-need-to-know))

| Гендер  | Процент (%) |
| --- | ------ |
| Мужчина | 56 |
| Женщина | 44 |



| Возраст (лет) | Процент пользователей (%) |
| --- | ------ |
| 13-17 |  6.6   |
| 18-24 |  17.1   |
| 25-34 |  38.5  |
| 35-49 |  20.7   |
| 50+  | 17.1   |

MVP функционал:
- Регистрация
- Аккаунт: профиль, информация о профиле
- Фото и видео для профиля
- твиты, фото и видео в твитах, репосты твитов
- комментарии к твитам и комментарии к комментариям(в виде дерева)
- рекомендательная система твитов
- Настройки профиля
- Поиск: поиск по хэштегам
- Уведомления

Ключевой функционал - посты и поиск по ним, переписка между пользователями.
Ключевые продуктовые решения:
- хэштеги: для поиска и группировки твитов по темам
- ретвиты

## 2. Расчет нагрузки

### Продуктовые метрики

| метрика | значение |
| --- | ------ |
| MAU |  550MM |
| DAU |  217MM |

| память на 1 | значение |
| --- | ------ |
| твит |  300 байт |
| фото | 16 кбайт |
| видео | 600 кБайт |


| действия пользователя (среднее) | значение | память |
| ---  | ---- |  --- |
| чтение твитов | 50 | - |
| лайк (твита и комментария) | 10 | 8 байт |
| поиск твита | 30 | - |
| написание комментария | 4 | 100 байт |
| написания твита | 2.4 | (расчитано ниже) |
| репост твита | 1.7 | - |

возьмём, что каждый 3 твит с видео, и в каждом твите в среднем есть 2 фотографии.

32 КБ + 200 Байт = 32.2 КБ (твит с 2 фото)
200 байт + 600 КБайт = 0.6002 МБ(твит с видео)
100 байт * 10 = 1000 байт ~ 1КБ (комментарии)

Средний размер хранилища пользователя - 532.176 КБ - пользователь/ сутки

### Технические метрики

хранения в разбивке по типам данных

| Данные | Кол-во в сутки | память в сутки (ГБ) |
| ---   |  -- |  ---              |
| твиты | 518.4 M |     145       |
| фото | 1 B |          15820     |
| видео | 172 M |       98420     |
| комментарии | 1.3 B | 81 |

| Сетевой трафик | среднее потребление | пиковое потребление |
| --- | ---- | --- |
| твиты | 0.001678  ГБ/c |  0.004195 ГБ/c  | 
| фото |  0.1831018 ГБ/c   |  0.4577545 ГБ/c  |
| видео | 1.13912037 ГБ/c   | 2.847800925  ГБ/c |

Суммарный суточный  трафик - 114466 ГБ/cутки

| RPS | средний | пик |
| ---  | ---- | --- |
| получение твита | 125787  | 314467 |
| лайк твита | 25157  | 62892 |
| поиск твита | 251574  | 628935 |
| написание комментария | 15000  | 37500 |
| написания твита | 6000 | 15000  | 
| репост твита | 4276  | 10691  |
| проверка авторизации + авторизация | 1000000 | 2500000 |
| изменение настроек профиля | 200 | 450 |
| подписка на / отписка от пользователя | 600 | 1500 |

## Глобальная балансировка нагрузки.

Разбиение по доменам.

x.com - единственный домен



| Расположение ДЦ | назначение |
| --- | -- |
| Атланта (Джорджия) | основной (США) |
| Портленде (Орегон) | основной (США) |
| Даллас (Техас) | для масштабирования (США) |
| Сингнапур | Азиатско-Тихоокеанский регион |
| Дублин (Ирландия) | Европа |

Такое расположение датацентров, потому что как показывает статистика [oberlo](https://www.oberlo.com/statistics/number-of-twitter-users-by-country "распределение по странам"), наибольшая часть аудитории "X" находится в США и Азии, после идёт Европа и Южная Америка. Такое расположениец ДЦ позволит ускорить доступ к сервису для большей части аудитории.

| Регион | % пользователей |
| --- | -- |
| Северная Америка | 35.62% |
| Азия | 46.20% |
| Европа | 11.90% |
| Южная Америка | 6.28% |

| RPS                                    | Атланта(Джорджия) | Портленд (Орегон) | Даллас(Техас) | Сингнапур | Дублин (Ирландия) |
| ---                                    | ----              | ---               | ---           | ---       | ---               |
| получение твита                        | 22,551            | 22,551            | 7,345         | 58,085    |     14,950        |
| лайк твита                             | 4,503	         | 4,503             |   1,467       | 11,624    |     2,994         |
| поиск твита                            | 45,033	         | 45,033            | 14,667        | 116,975   |  30,440           |
| написание комментария                  | 2,684             | 2,684             | 874           | 6,930     | 1,785             |
| написания твита                        | 1,073             | 1,073             | 349           | 2,772     | 714               |
| репост твита                           | 765               | 765               | 249           | 1,974     | 509               |
| проверка авторизации + авторизация     | 179,917           | 179,917           | 58,172        | 461,946   | 119,048           |	
| изменение настроек профиля             | 36	             | 36                | 12            | 92        | 24                |
| подписка на / отписка от пользователя  | 108               | 108               | 35            | 277       | 71                |

DNS балансировка - Latency-based DNS \
Anycast балансировка - BGP Anycast

### Список источников:
1. [X](https://x.com/ "сам твиттер")
2. [Statista](https://www.statista.com/statistics/242606/number-of-active-twitter-users-in-selected-countries/)
3. [Tridenstechnology](https://tridenstechnology.com/ru/c%D1%82%D0%B0%D1%82%D0%B8%D1%81%D1%82%D0%B8%D0%BA%D0%B0-%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D0%B5%D0%B9-twitter/)
4. [MeltWater](https://www.meltwater.com/en/blog/twitter-stats-marketers-need-to-know)
5. [wifitalents](https://wifitalents.com/statistic/twitter/ "немного отличается от предыдущих статистик")
6. [quora](https://www.quora.com/Does-Twitter-store-its-data-and-if-so-where-and-how-is-it-organized#:~:text=By%20means%20of%20Hadoop%2C%20Twitter,Twitter%20is%20over%2010K%20nodes "немного инфы про RPS твиттера")
7. [oberlo](https://www.oberlo.com/statistics/number-of-twitter-users-by-country "распределение по странам")
8. [siliconangle](https://siliconangle.com/2022/12/30/twitter-reportedly-closes-sacramento-data-center-part-cost-cutting-initiative/ "где главные датацентры твиттера")
9. [servernews](https://servernews.ru/1077720 "где главные датацентры твиттера")
