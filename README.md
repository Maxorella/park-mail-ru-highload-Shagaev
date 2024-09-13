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

(предположения - точной информации нету)




| память на 1 | значение |
| --- | ------ |
| твит |  300 байт |
| фото | 1 мбайт |
| видео |  20 мбайт |


| действий пользователя (среднее) | значение |
| ---  | ---- |
| чтение твитов | 400 |
| лайк | 50 | 
| поиск твита | 50 |
| написание комментария | 10 |
| написания твита | 2 |
| репост твита | 5 |

Средний размер хранилища пользователя -  30 мбайт - пользователь/ сутки

### Технические метрики

хранения в разбивке по типам данных

| Данные | Гбайт |
| --- | ---- |
| твиты | 450MM | 128746 | 
| фото | 300MM | 292968 |
| видео | 100MM | 5859360 |

| Сетевой трафик | пиковое потребление |
| --- | ---- |
| твиты | 0.15  ГБ/c | 
| фото |  0.34 ГБ/c | 
| видео | 6.78 ГБ/c  | 

Суммарный суточный  трафик - 6281074 ГБ/c

| RPS | средний | пик |
| ---  | ---- | --- |
| чтение твитов | 995  | 2500 |
| лайк | 124  | 310  |
| поиск твита | 124  | 310  |
| написание комментария | 25  | 62 |
| написания твита | 5 | 13  | 
| репост твита | 12  | 30  |


### Список источников:
1. [X](https://x.com/ "сам твиттер")
2. [Statista](https://www.statista.com/statistics/242606/number-of-active-twitter-users-in-selected-countries/)
3. [Tridenstechnology](https://tridenstechnology.com/ru/c%D1%82%D0%B0%D1%82%D0%B8%D1%81%D1%82%D0%B8%D0%BA%D0%B0-%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D0%B5%D0%B9-twitter/)
4. [MeltWater](https://www.meltwater.com/en/blog/twitter-stats-marketers-need-to-know)
5. [wifitalents](https://wifitalents.com/statistic/twitter/ "немного отличается от предыдущих статистик")
